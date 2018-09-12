
## Description

PM2 module to automatically rotate logs of processes managed by PM2 and upload to s3 before deleting old logs.

## Install

`pm2 install pm2-logrotate-s3-upload`

## Configure
create pm2-logrotate-s3-upload-config.js file in PM2 home folder which by default is ~/.pm2 .
set config values as given in example below:

```javascript
module.exports = {
                    "max_size": "1KB",
                    "retain": "2",
                    "compress": true,
                    "dateFormat": "YYYY-MM-DD_HH-mm-ss",
                    "workerInterval": "30",
                    "rotateInterval": "0 0 * * *",
                    "rotateModule": true,
                    "aws": {
                      "credentials": {
                        "accessKeyId": "<AWS_ACCESS_KEY_ID>",
                        "secretAccessKey": "<AWS_SECRET_ACCESS_KEY>",
                        "region": "<AWS_REGION>"
                      }
                    },
                    "logBucketSetting": {
                      "bucket": "<S3_BUCKET>",
                      "s3Path": "website-node-react",
                      "s3FilePathFormat": "__year__/__month__/__day__/__ip__/__filename__"
                    },
                    "getAWSPublicIp": true,
                    "serverIp": null
                }

```

### Config Property Description
#### S3 upload properties
All of the following properties needs to defined for s3 upload to work.
- `aws.credentials.accessKeyId` (Defaults to `null`): This is access key id of your aws account.
- `aws.credentials.secretAccessKey` (Defaults to `null`): This is secret key of your aws account.
- `logBucketSetting.bucket` (Defaults to `null`): This is region of your s3 in aws.
- `logBucketSetting.s3Path` (Defaults to `null`): This is s3 path in bucket where all your logs would be kept.
- `logBucketSetting.s3FilePathFormat` (Defaults to `__year__/__month__/__day__/__ip__/__filename__`): This is key path where your logs would be uploaded. Here `__year__`, `__month__`, `__day__`, `__ip__`, `__filename__`, and `__epoch__` are place holders for dynamic value to be inserted in key path.
One of two properties should be set for this to work.
- `getAWSPublicIp` (Defaults to `false`): This extracts the public ip of ec2 instance from aws meta data api and set the key path.
- `serverIp` (Defaults to `null`): This is the static value of ip of server that needed to be passed.

#### PM2 log rotate properties
- `max_size` (Defaults to `10MB`): When a file size becomes higher than this value it will rotate it (its possible that the worker check the file after it actually pass the limit) . You can specify the unit at then end: `10G`, `10M`, `10K`
- `retain` (Defaults to `all`): This number is the number of rotated logs that are keep at any one time, it means that if you have retain = 7 you will have at most 7 rotated logs and your current one.
- `compress` (Defaults to `false`): Enable compression via gzip for all rotated logs
- `dateFormat` (Defaults to `YYYY-MM-DD_HH-mm-ss`) : Format of the data used the name the file of log
- `rotateModule` (Defaults to `true`) : Rotate the log of pm2's module like other apps
- `workerInterval` (Defaults to `30` in secs) : You can control at which interval the worker is checking the log's size (minimum is `1`)
- `rotateInterval` (Defaults to `0 0 * * *` everyday at midnight): This cron is used to a force rotate when executed.
We are using [node-schedule](https://github.com/node-schedule/node-schedule) to schedule cron, so all valid cron for [node-schedule](https://github.com/node-schedule/node-schedule) is valid cron for this option. Cron style :
```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    |
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)
```
