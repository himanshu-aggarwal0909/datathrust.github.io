---
layout: post
title: Changing S3 Storage Class (Archiving the data to glacier) using Python
# author: "Himanshu Aggarwal"
# description: "Python utility to change the storage classes of all the objects present in a s3 directory."
date: 2022-03-01
categories: data engineer code wall
---


```python
import argparse
import sys
from datetime import datetime, timedelta
import os
import logging
import boto3
from urllib.parse import urlparse
from botocore.exceptions import ClientError
import subprocess


class ManualArchive():

	s3_client = boto3.client('s3') 

	def __init__(self, table_name, table_path, partition_column, start_date_str, end_date_str, retention_days, batch_size, static_partition_present, resume_from_checkpoint):
		"""
		Initializes the ManualArchive Object

		:param:  table_name
		:param:  table_path
		:param:  partition_column
		:param:  start_date_str
		:param:  end_date_str
		:param:  retention_days
		:param:  batch_size
		:param:  static_partition_present
		:param:  resume_from_checkpoint
		"""
		self.table_name = table_name
		self.table_path = table_path
		self.partition_column = partition_column
		self.start_date_str = start_date_str
		self.end_date_str = end_date_str
		self.retention_days = retention_days 
		self.batch_size = batch_size 
		self.static_partition_present = static_partition_present
		self.resume_from_checkpoint = resume_from_checkpoint
		self.log_file = '{}_logs_{}'.format(args.table_name.replace("-", "_"), datetime.now().strftime("%Y_%m_%d_%I_%M"))
		

	def _get_bucket_prefix(self, curr_table_path):
		"""
		This method provides the bucket name and prefix from the passed path
		:param:  s3_path
		:return: bucket_name, prefix_name
		"""
		table_path = urlparse(curr_table_path, allow_fragments=False)
		return table_path.netloc, table_path.path.strip("/")


	def _get_all_objects(self, s3_path):
		"""
		Return all objects in the s3 directory 
		:param: bucket_name
		:param: prefix_name
		
		:return: list of all the objects
		"""
		bucket_name, prefix_name = self._get_bucket_prefix(s3_path) 
		try:
			response = []
			paginator = ManualArchive.s3_client.get_paginator('list_objects_v2')
			pages = paginator.paginate(Bucket=bucket_name, Prefix=prefix_name)
			for page in pages:
				if 'Contents' in page:
					response += page['Contents']
				else:
					logging.info("Error in _get_all_objects Contents key not found response : {}".format(response))
					return False
			return response
		except Exception as e:
			logging.exception("Exception occurred _get_all_objects Error message : {}".format(str(e)))
			return False
	

	def _change_storage_class(self, bucket_name, object_key, handle_invalidobjectstate=True):
		"""
		This method changes the storage class of the pass object
		:param: bucket_name str
		:param: object_key str
		:param: handle_invalidobjectstate handles the invalid object state when storage class is already set to glacier

		:return: result boolean 
		"""
		copy_source = {
			'Bucket': bucket_name,
			'Key': object_key
		}
		try:
			ManualArchive.s3_client.copy(
				copy_source, bucket_name, object_key,
				ExtraArgs = {
					'StorageClass': 'GLACIER',
					'MetadataDirective': 'COPY'}
			)
			return True
		except ClientError as e:
			if e.response['Error']['Code'] == 'InvalidObjectState' and handle_invalidobjectstate and e.response['Error']['StorageClass'] == 'GLACIER':
				logging.info(e.response['Error'])
				return True
			else:
				logging.exception("Exception occurred while changing storage class object_key : {}, Error: {}".format(object_key, str(e)))
				self._push_log_file()
				raise e


	def _change_storage_class_dir(self, curr_table_path):
		"""
		This function provides the bucket name and prefix from the passed path
		:param:  s3_path
		:return: bucket_name, prefix_name
		"""
		logging.info("changing storage class for path : {}".format(curr_table_path))
		all_objects = self._get_all_objects(curr_table_path)
		if all_objects:
			for each_obj in all_objects:
				obj_key = each_obj['Key']
				logging.debug("changing storage class for object : {}".format(obj_key))
				bucket_name, prefix_name = self._get_bucket_prefix(curr_table_path)
				rs = self._change_storage_class(bucket_name, obj_key)
				logging.debug("result : {}".format(rs))
				if not rs:
					return False
					break
		else:
			return False
		return True


	def _create_checkpoint(self, date_passed_str):
		"""
		This function creates the checkpoint file for the specified table_name in the local current dir and appends the last run date.
		:param: table_name str
		:param: date_passed_str str

		:return: None
		"""
		file = open("{}.csv".format(self.table_name), "a")
		file.write(date_passed_str + "\n")
		file.close()


	def _fetch_date_from_checkpoint(self):
		"""
		This function fetches the last run date from the checkpoint file.
		:param: table_name str

		:return: last_run_date datetime
		"""
		logging.info("Fetching last checkpoint date for table_name : {}".format(self.table_name))
		try:
			file = open("{}.csv".format(self.table_name), "r")
			last_date_str = file.readlines()[-1].replace("\n", "").strip()
			file.close()
			logging.info("last_date_str : {}".format(last_date_str))
		except Exception as e:
			logging.exception("Exception occurred while getting the checkpoint date!! Error message : {}".format(str(e)))
			self._push_log_file()
			raise e
		return datetime.strptime(last_date_str, "%Y-%m-%d")


	def _manual_archive(self):
		"""
		This function is used to archive all the objects present in the current directory.
		
		:return: None (Exists the program if the job is failed)
		"""
		logging.info("Passed_arguments : {}".format(str([self.table_name, self.table_path, self.partition_column, self.start_date_str, self.end_date_str, self.retention_days, self.batch_size, self.static_partition_present, self.resume_from_checkpoint])))
		start_date = self._fetch_date_from_checkpoint() if self.resume_from_checkpoint else datetime.strptime(self.start_date_str, "%Y-%m-%d")
		end_date = (datetime.now() - timedelta(self.retention_days + 1)) if self.end_date_str is None else datetime.strptime(self.end_date_str, "%Y-%m-%d")
		logging.info("start_date : {}\nend_date: {}".format(start_date, end_date))
		total_days_run = (end_date - start_date).days
		logging.info("total_days_run : {}".format(total_days_run))
		for each in range(total_days_run):
			current_date = start_date + timedelta(each)
			current_date_str = datetime.strftime(current_date, "%Y-%m-%d")
			curr_table_path =  self.table_path.replace("RUN_DATE", current_date_str) if self.static_partition_present else "{}/{}={}".format(self.table_path.strip("/"), self.partition_column, current_date_str)
			logging.info("Changing storage class for Path {}".format(curr_table_path))
			result = self._change_storage_class_dir(curr_table_path)
			if not result:
				logging.error("Job Failed!!")
				return False
			else:
				self._create_checkpoint(current_date_str)
				logging.info("Successfully Changed storage class for Path {}".format(curr_table_path))
		return True


	def _setup_logging(self):
		"""
		This method setups the logging
		:return:
		"""
		logging.basicConfig( filename = self.log_file,filemode = 'a',level = logging.INFO,format = '%(asctime)s - %(levelname)s: %(message)s',datefmt = '%m/%d/%Y %I:%M:%S %p' )


	def _push_log_file(self):
		"""
		This method pushes the log file to s3
		"""
		logging.info("Pushing the log file to s3")
		s3_log_path = "s3://data-science-common-data-platform/dp-util-logs/{}/{}".format(self.__class__.__name__, self.log_file)
		command = "aws s3 cp {} {}".format(self.log_file, s3_log_path)
		status, output = subprocess.getstatusoutput(command)
		if status != 0:
			raise Exception("Unable to push data file")
		else:
			logging.info("Pushed the log file to s3 path : {}".format(s3_log_path))


	def main(self):
		self._setup_logging()
		self._manual_archive()
		self._push_log_file()

if __name__ == "__main__":
	parser = argparse.ArgumentParser()
	parser.add_argument("--table-name", dest="table_name", required=True, type=str)
	parser.add_argument("--table-path", dest="table_path", required=True, type=str)
	parser.add_argument("--partition-column", dest="partition_column", required=True, type=str)
	parser.add_argument("--start-date-str", dest="start_date_str", required=False, type=str)
	parser.add_argument("--end-date-str", dest="end_date_str", required=True, type=str)
	parser.add_argument("--retention-days", dest="retention_days", required=True, type=int)
	parser.add_argument("--batch-size", dest="batch_size", required=True, type=int)
	parser.add_argument("--static-partition-present", dest="static_partition_present", action='store_true', default=False)
	parser.add_argument("--resume-from-checkpoint",dest="resume_from_checkpoint", action='store_true', default=False)
	args = parser.parse_args()
	manual_archive_obj = ManualArchive(args.table_name, args.table_path, args.partition_column, args.start_date_str, args.end_date_str, args.retention_days, args.batch_size, args.static_partition_present, args.resume_from_checkpoint)
	manual_archive_obj.main()
```

```sh
python3 ManualArchive.py --table-name "glacier-test-final" --table-path "s3://glacier-test-final/day=2021-07-01/" --partition-column "day" --start-date-str "2021-07-10" --end-date-str "2021-07-12" --retention-days 10 --batch-size 10
```