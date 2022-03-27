---
layout: post
title: Running Azkaban Jobs Using Python
# author: "Himanshu Aggarwal"
# description: "Python utility to Run Selective Azkaban Jobs."
date: 2022-03-01
categories: data engineer code wall
---


Azkaban is a workflow orchestrator, used to schedule big data jobs.

Pipeline are made up of a series of steps implemented in a specific order to process data and transfer it from one system to another.
Over the time, as the business usecases increases, large number of jobs keeps on adding in the pipeline and the directed acyclic graph(DAG) of jobs gets complex and messy.

<figure>
<img src="{{ site.url }}/images/azkaban_complex_dags.png"/>
<figcaption align = "center"><i>Azkaban Complex Job DAG</i></figcaption>
</figure>

Executing selective jobs from the above job DAGs becomes very cumbersome.

So, we can use the requests python module to run/automate only selective jobs from the DAG.

```python
import requests
import argparse
import logging
import sys
import traceback


# Close the warning returned by calling the api request
# requests.packages.urllib3.disable_warnings()

# Define azkaban address, login information
str_url = 'http://a.b.c.d:8081'


class ParseKwargs(argparse.Action):
    def __call__(self, parser, namespace, values, option_string=None):
        setattr(namespace, self.dest, dict())
        for value in values:
            key, value = value.split('=')
            getattr(namespace, self.dest)[key] = value

def parse_args():
    parser = argparse.ArgumentParser()
    # To do : Need to restore azkaban credentials in kms 
    parser.add_argument("--username", help="azkaban username", type=str, required=True)
    parser.add_argument("--password", help="azkaban password", type=str, required=True)
    parser.add_argument("--projectName", help="azkaban flowname to execute", type=str, required=True)
    parser.add_argument("--flowName", help="azkaban flowname to execute", type=str, required=True)
    parser.add_argument("--listOfJobs", help='comma separeted List of jobs to be executed e.g. job_a,job_b', type=str, default="")
    parser.add_argument("--flowParameters", help='Azkaban flow parameters e.g. {"flow_start_date": "2021-06-01"}', nargs='*', action=ParseKwargs, default="")
    parser.add_argument("--azkabanSettings", help='Azkaban flow parameters e.g. {"concurrentOption":"ignore"}', nargs='*', action=ParseKwargs, default="")

    return parser.parse_args()


def login(username, password):
    """
    request to login.

    :parameters username , password
    :returns request with logged in
    """
    postdata = {'username':username,'password':password}
         # , by verify=False closes security verification
    login_url = str_url + '?action=login'    
    r = requests.post(login_url, postdata, verify=False).json()
    return r


def fetch_jobs(sessionid, projectname, flowid):
    """
    Fetches all the jobs along with each dependency.

    :parameters sessionid   : current logged in session id
                projectname : azkaban project name
                flowid      : azkaban flow name

    :return
    all the jobs along with dependency (as request output in json)
    """
    postdata = {'session.id':sessionid, 'project':projectname, 'ajax':'fetchflowgraph', 'flow':flowid}
    fetch_url = str_url + '/manager?ajax=fetchflowgraph'
    r = requests.get(fetch_url, postdata, verify=False).json()
    return r


def execute_flow(session_id, project_name, flow_id, settings):
    """
    Execute the azkaban flow

    :parameters session_id   : current logged in session id
                project_name : azkaban project name
                flow_id      : azkaban flow name
                settings     : parameter dictionary

    :return
    request object (with status and execution id)
    """
    postdata = {'session.id': session_id, 'ajax': 'executeFlow', 'project': project_name, 'flow': flow_id}
    for key in settings:
        postdata[key] = settings[key]
    fetch_url = str_url + '/executor?ajax=executeFlow'
    r = requests.get(fetch_url, postdata, verify=False).json()
    return r


def get_disabled_job_list(session_id, project_name, flow_id, jobs_to_be_executed):
    """
    Getting the list of the jobs that should be dislabled 
    (i.e. all jobs except jobs to be executed)
    
    :parameters session_id          : current logged in session id
                project_name        : azkaban project name
                flow_id             : azkaban flow name
                jobs_to_be_executed : Jobs list 

    :return     
    list of jobs to be disabled (e.g.  '["job_a","job_b"]')
    """
    jobs_list = fetch_jobs(session_id, project_name, flow_id)
    disabled_jobs_set= set([each['id'] for each in jobs_list['nodes']]) - set(jobs_to_be_executed)
    return "[{}]".format(",".join('"{}"'.format(i) for i in disabled_jobs_set))


def execute_flow_with_selected_jobs(session_id, project_name, flow_id, jobs_to_be_executed, override_parameters, additional_settings_params={}):
    """
    :parameters session_id                 : current logged in session id
                project_name               : azkaban project name
                flow_id                    : azkaban flow name 
                jobs_to_be_executed        : list of jobs to execute
                override_parameters        : dict of the parameters to overwrite 
                                             the default azkaban properties.
                additional_settings_params : dict of settings (mentioned in comments)
    
    :return
    request object (with status and execution id)
    """
    #preparing the setting for the executeFlow call
    settingss = {}
    for each_key, each_val in override_parameters.items():
        settingss["flowOverride[{}]".format(each_key)] = each_val
    for each_key, each_val in additional_settings_params.items():
        settingss[each_key] = each_val
    #Getting the list of disabled Jobs
    logging.info("jobs_to_be_executed : {}".format(str(jobs_to_be_executed)))
    disabled_jobs = get_disabled_job_list(session_id, project_name, flow_id, jobs_to_be_executed)
    settingss["disabled"] = disabled_jobs
    logging.info("settings {}:".format(str(settingss)))
    return execute_flow(session_id, project_name, flow_id, settingss)


if __name__ == '__main__':
    logging.basicConfig(format='%(asctime)s : %(lineno)d - %(message)s', level = logging.INFO)
    args = parse_args()
    logging.info("Passed Arguments {}".format(str(args)))
    try:
        r_login = login(args.username, args.password)
    except Exception as e:
        logging.error("Exception Occurred in login : {}".format(str(traceback.format_exc())))
        sys.exit(1)
    logging.info("Login request object : {}".format(str(r_login)))
    if r_login.get('status') == 'success':
        logging.info("Logged in successfully login")
        session_id = r_login['session.id']
        list_of_jobs = args.listOfJobs.split(",")
        try:
            r_execute = execute_flow_with_selected_jobs(session_id, args.projectName, args.flowName, list_of_jobs, args.flowParameters, args.azkabanSettings)
            logging.info("Execution Requestion Object : {}".format(str(r_execute)))
            if r_execute.get('execid') > 0:
                logging.info("Successfully started the execution of the flow!!")
            else:
                logging.error("Error Occurred in executing the flow : {}".format())
        except Exception as e:
            logging.error("Exception Occurred in execute_flow_with_selected_jobs : {}".format(str(traceback.format_exc())))
            sys.exit(1)
    else:
        logging.error("Error Occurred at login : {}".format(r_login.get('error')))


## Sample Usage : 
#python3 azkabanUtility.py --username=<username> --password=<password> --projectName azkaban-job-demo-test --flowName=dp-der-end-worflow --flowParameters flow_start_date=2021-06-01 --azkabanSettings concurrentOption=ignore --listOfJobs='dp-der-job-1,dp-der-job-2'

#   settings   :Optional parameter dictionary
#   disabled(Optional):This execution performs a list of disabled job names. Formatted asJSONArray string. Such as:["job_name_1","job_name_2","job_name_N"]
#   successEmails(Optional):Perform a successful mailing list. Multiple mailboxes[,|;|\s+]Separate. Such as: zh@163.com,zh@126.com
#   failureEmails(Optional):Perform a successful mailing list. Multiple mailboxes[,|;|\s+]Separate. Such as: zh@163.com,zh@126.com
#   successEmailsOverride(Optional):Whether to overwrite a successful message with the system default email settings. value:true, false
#   failureEmailsOverride(Optional):Whether to overwrite failed messages using the system default email settings. value:true, false
#   notifyFailureFirst(Optional):Whether to send an email notification on the first failure. value:true, false
#   notifyFailureLast(Optional):Whether to send an email notification when the last failure. value:true, false
#   failureAction(Optional):If something goes wrong, how to do it. Value: finishcurrent, cancelimmediately, finishpossible
#   concurrentOption(Optional):If you don't need any details, use ignore. Value: ignore, pipeline, skip
#   flowOverride[flowProperty](Optional):Overrides the specified stream property with the specified value. Such as: flowoverride[failure.email]=abc@163.com
```

To use the same as a python utility, save the above script as .py file and call the script by passing the arguments.

For, example 

```sh
python3 azkabanUtility.py --username=<username> --password=<password> --projectName azkaban-job-demo-test --flowName=dp-der-end-worflow --flowParameters flow_start_date=2021-06-01 --azkabanSettings concurrentOption=ignore --listOfJobs='dp-der-job-1,dp-der-job-2'
```