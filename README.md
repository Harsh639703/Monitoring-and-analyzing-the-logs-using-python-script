# Monitoring-and-analyzing-the-logs-using-python-script

import re
import pandas as pd
import logging
import time

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Regular expression pattern to match IP addresses
pattern = r"((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"

# Lists to store parsed data
ip_addrs_lst = []
failed_lst = []
success_lst = []
statuscode = []
total_rows = 0

while True:
    try:
        try:
# Opening  the log file for reading
            with open("serverlogs.log", "r") as logfile:
# here we are reading  all lines into a list
                lines = logfile.readlines()
        except FileNotFoundError:
            logging.error("Log file not found.")
            exit(1)

# we are finding if there are new lines are available in the log file or not
        if total_rows < len(lines):
# here we are iterating  the each line in the log file
            for log in lines:
# here we are searching for an IP address pattern in the line
                ip_add = re.search(pattern, log)
                if ip_add:
# If IP address found, we are  appending that to the list
                    ip_addrs_lst.append(ip_add.group())
# Split the line by space to extract relevant data
                    lst = log.split(" ")
# in this  the total number of rows processed is increasing
                    total_rows += 1
                    try:
# here we are extracting  and convert relevant data
                        failed_lst.append(int(lst[-1]))
                        success_lst.append(int(lst[-4]))
                        statuscode.append(int(lst[8]))
                    except (IndexError, ValueError):
# here we are handling  errors due to invalid log entry format
                        logging.error("Invalid log entry format")

# here we are checking  if any valid log entries were found
        if not ip_addrs_lst:
            logging.error("No valid log entries found.")
            exit(1)

# here we are calculating the  total success and failure counts
        total_failed = sum(failed_lst)
        total_success = sum(success_lst)

# here we are creating  a dataframe to store parsed data
        df = pd.DataFrame(columns=['IP Address', "Success", "Failed", "Statuscode"])
        df['IP Address'] = ip_addrs_lst
        df["Success"] = success_lst
        df["Failed"] = failed_lst
        df["Statuscode"] = statuscode

# here we are converting the  parsed data to CSV file
        try:
            df.to_csv("output.csv", index=False)
        except PermissionError:
            logging.error("Permission denied. Cannot write to file.")
            exit(1)


        logging.info(df)
        logging.info("Length of HTTP status code is: %d", len(statuscode))

# here it is generating the  log error report
        error_report = df[df['Statuscode'] >= 400]
        error_report.to_csv("error_report.csv", index=False)
        logging.info("\nError Report:")
        logging.info(error_report)
        logging.info("Total Failed: %d", total_failed)
        logging.info("Total Success: %d", total_success)

# after processing the all logs it will go in sleep mode and wait for the other log to process them       
        logging.info("Sleeping for 10 secs")
        time.sleep(10)
# here we are handling the keyboard interupt
    except KeyboardInterrupt:
        
        logging.info("Task Ended")
        break

# here this message is showing completion of the all task
logging.info("task done")
