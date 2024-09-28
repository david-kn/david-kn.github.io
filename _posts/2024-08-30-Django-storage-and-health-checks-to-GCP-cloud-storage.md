---
title: "Django-health-check plugin shows problem with GCP cloud storage with django-storage"
date: 2024-08-30
tags: python
---

Configuring a Django application to manage external file storage on Google Cloud Platform (GCP) initially surprised me. I was relieved to discover that after a quick setup, a few straightforward configurations in both GCP and the Django `settings.py` file, there was a working workflow. Django began uploading data to GCP, making the files immediately accessible and downloadable. However, the issue arose during a health check, which reported that the cloud storage was unavailable and displayed a very generic exception message:  ```ServiceUnavailable: Unknown exception```

Looking into the logs I was able to find a bit more info:

```
Traceback (most recent call last):
File "/usr/local/lib/python3.10/site-packages/health_check/storage/backends.py", line 75, in check_status
  file_name = self.check_save(file_name, file_content)
File "/usr/local/lib/python3.10/site-packages/health_check/storage/backends.py", line 57, in check_save
  raise ServiceUnavailable("File does not exist")
health_check.exceptions.ServiceUnavailable: unavailable: File does not exist

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
File "/usr/local/lib/python3.10/site-packages/health_check/backends.py", line 30, in run_check
  self.check_status()
File "/usr/local/lib/python3.10/site-packages/health_check/storage/backends.py", line 79, in check_status
  raise ServiceUnavailable("Unknown exception") from e
health_check.exceptions.ServiceUnavailable: unavailable: Unknown exception
```


I reviewed both the GCP logs and library documentation (as well as related issues on GitHub) and realized I had overlooked some important information (oups) regarding a specific bucket type in Google Cloud. Based on our organizational settings, my storage bucket was `uniform` with no chance to switch to `fine-grained`. Therefore, as mentioned in [django-storage docs](https://django-storages.readthedocs.io/en/latest/backends/gcloud.html) I tried to set appropriatelly attributes `GS_QUERYSTRING_AUTH` and `GS_DEFAULT_ACL`. Moving away from the default configuration, where I had only provided the bucket ID and JSON credentials, I updated the attributes accordingly with:

```
    "GS_DEFAULT_ACL" : None,
    "GS_QUERYSTRING_AUTH": False,
```

However, after failing to succeed with this settings as well, I "experimented" with the parameters and eventually found that the __opposite value__ needed to be __explicitly__ set to make it work! (even though docs describe `True` as a defaul value). I didn’t delve deeper into the library’s source code about, but this was the only solution that worked (and rule from the Zen of Python _Explicit is better than implicit_ would make a smile on my face if it didn't take me quite some time to find this solution a bit by chance).

```
  "GS_QUERYSTRING_AUTH": True,
```
Finally, both actual upload and health-check are working.

![Screenshot](/assets/images/2024-08-30-Django-storage-status-OK.png) *All statuses are OK now*

A configuration related to GCP storage with uniform bucket type and Django:

```
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.gcloud.GoogleCloudStorage",
        "OPTIONS": {
          "GS_BUCKET_NAME": "bucket-test-name",
          "GS_PROJECT_ID" : "123456",
          "GS_CREDENTIALS": service_account.Credentials.from_service_account_file("./secrets/gcp-bucket-creds.json"),
          "GS_QUERYSTRING_AUTH": True,
          "GS_FILE_OVERWRITE": False,
        },
    },
}
```
