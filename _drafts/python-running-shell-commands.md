---
layout: post
title: Running bash shell commands using python.
# author: "Himanshu Aggarwal"
# description: "Python utility to Run Bash Commands and getting output at runtime as well."
date: 2022-03-01
categories: python tutorials
---

```python

#displays getting realtime output using subprocess
def run_subprocess_command(command, purpose, symbol="*", show_output):
	logging.info("Running Command : {}".format(command))
	command_process = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)	
	while command_process.poll() is None:
		line = command_process.stderr.readline()
		if show_output:
			print(line.strip())
			log_file.write(line.strip().decode("utf-8")+"\n")
	command_process.wait()
	result = True if command_process.poll() == 0 else False
	rs_stdout_byte, rs_stderr_byte = command_process.communicate()
	rs_stdout = rs_stdout_byte.decode("utf-8")
	rs_stderr = rs_stderr_byte.decode("utf-8")
	if show_output:
		print("Result : ", result)
		print("Stdout : ", rs_stdout)
		print("Stderr : ", str(rs_stderr).replace("\n", " \n "))
		print(symbol*(len(purpose)+30), end=" \n \n")
	log_file.write("Result : "+ str(result) +"\n" +"Stdout : "+ str(rs_stdout)+"\n" +"Stderr : "+ str(rs_stderr) + " \n " + symbol*(len(purpose)+30)+" \n \n")
	return result, rs_stdout, rs_stderr 

```