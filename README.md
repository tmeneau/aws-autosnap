# Dockerized aws-autosnap
aws-autosnap is a python script to make it easy to *automatically snapshot any EBS volumes*.

Simply add a tag to each instance/volume you want to snapshots regularly, configure and run this container and you're done. It will even handle removing old snapshots on a hourly based on a retention policy.

Features:
- *Python based*: Leverages boto and is easy to configure.
- *Tag-based configuration*: Instance/volume specific settings are set using tags directly on those objects.
- *Flexible frequency/retention policy*: Specify snapshot frequency and snapshot retention using either the default value in config file or tags(instance or volume)
- *SNS Notifications*: Autosnap works with Amazon SNS out of the box, so you can be notified of created/deleted of snapshots

## Usage

### Authentication
You'll need to give autosnap the correct permissions on your AWS account in order to function. You can use either an IAM user or role. Refer to the [sample IAM policy](iam.policy.sample) when making your IAM policy attached to this user/role. If you're not using SNS notifications, you can remove that portion.

* If you're using an IAM user, you must set the access and secret keys in the config file, or as [environment variables](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-environment)(`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`).
* If you're using an EC2 role, just run the script! It'll authenticate automatically.

### SNS (optional)
If you'd like to use SNS notifications, create an SNS topic in your AWS account and pass in the ARN in the config file, or as environment variable `AUTOSNAP_SNS_ARN`.

### Configuration & Run
All the configuration parameters can be *overridden* by passing environment variables. The variable naming convention is as follows, AUTOSNAP_{VARIABLE_NAME}. So, for example AUTOSNAP_LOG_FILE.
#### Examples
1. Run with SNS enabled
	```docker run -d --name ebs-snapshot-service -e AWS_ACCESS_KEY_ID=[CHANGE_ME] -e AWS_SECRET_ACCESS_KEY=[CHANGE_ME] -e AUTOSNAP_SNS_ARN=[CHANGE_ME] xetusoss/ebs-snapshot-service```
2. Dry run
	```docker run -d --name ebs-snapshot-service -e AWS_ACCESS_KEY_ID=[CHANGE_ME] -e AWS_SECRET_ACCESS_KEY=[CHANGE_ME] -e AUTOSNAP_DRY_RUN=true xetusoss/ebs-snapshot-service```

### Snapshot Scheduling
For each _instance_ or _volume_ that you want to snapshot, add the following tags:
  * (required) `autosnap:X`: how often (in hours) you want this instance to be snapshotted (tag name can be changed in `config.py`)
  * (optional) `autosnap_retention:X`: how many snapshots you want to keep (if not specified, it will use the value in `config.py`.
  * (optional) Tag any _volumes_ that you don't want to snapshot with `autsnap_ignore`. The tag's value doesn't matter (it can be blank).
  
  **PLEASE NOTE: _volume_ settings will *override* _instance_ settings**


### What happens
When this script creates a snapshot, it will tag the snapshot with `snapshot_type:autosnap` (or whatever `tag_name` is set to in `config.py`), along with some other useful tags. Later, when it is creating the list of snapshots to delete, it will only consider snapshots for a given volume if that tag is present. This allows you to make your own snapshots without having to worry about autosnap deleting them later (just make sure you don't tag it with 'snapshot_type:autosnap').
