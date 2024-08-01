# Django - Sending static files to AWS S3

This is a **basic** procedure about storing our static files in AWS Simple Storage System (S3).

As we know, in our development environment (DEBUG=True) Django can help us serving the static files necessary to run our application. But, when we deploy our application and start in the production environment (DEBUG=False), Django doesn't serve them for us anymore, and that's why we need something like AWS S3.

To follow this tutorial you have to use Django 4.2 or higher, since it was when the [STORAGES](https://docs.djangoproject.com/en/5.0/ref/settings/#storages) setting was added. [More details](https://docs.djangoproject.com/en/dev/releases/4.2/#custom-file-storages).

#### August 1, 2024

# Required libraries

- [python-decouple](https://pypi.org/project/python-decouple/)
- [django-storages[s3]](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html)

## Create an AWS account

First, you'll need to create an AWS account and use it to login in the AWS console. This link might help:
- https://signin.aws.amazon.com/signup?request_type=register

## Create an IAM user

IAM, or Identity Access Management, is a web server that helps us to control access to AWS resources. 

When we create an AWS account, we begin with a single identity (root user) that has access to all AWS resources in the account. But using this account for everyday tasks might not be safe, so AWS advises us to only use this root user for tasks that only it can perform. And then we can use IAM to create new identities and grant them only the permissions and accesses required for the tasks we want them to execute. 

To create the IAM user: 
- find the AWS service called IAM, go to 'Users' section and click in 'Create User';
- define a user name and move on. It's nice to have a name similar to the project you are using this user for, so you can easily relate them;
- we are not adding this new user to groups or copying any permissions. We just need to 'Attach policies directly' to it. So when you see this option you can click, choose a policy named 'AmazonS3FullAccess' and click 'Next';
- to finish you'll be asked to review your choices. You can just review and then finish the process.

## Create an access key for the new user

Once you have your new IAM user you can create an access key for it. 
- Again on the 'Users' section, click in the new user and find the option 'Create access key';
- you will be asked to choose a use case, but we can just choose 'Other' at this moment and move on;
- you will also be asked to write a description to this new access key, and this is optional. Write what best fits you and then finish by clicking in 'Create access key'.

Once you finish, you can save you 'Access key' and the 'Secret access key'. This will be the only chance you have, bacause later you won't be able to access them again. Store it in a safe place, and later we will use them to set up our AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables.

## Create an S3 bucket

Now we have to create an S3 bucket, which is where our files will be stored.
- find the AWS service called S3, and then click 'Create bucket' when you see the option;
- define a name for the bucket. It's nice to choose a name similar to the IAM user name we created before, it makes things easier to be managed. Also, this is the name we are going to use to set up our AWS_STORAGE_BUCKET_NAME environment variable;
- once the name is defined you can just skip the next options and click 'Create bucket'.

## Attach a new policy to the new S3 bucket

With our new bucket created, we have to attach a policy to it. This policy is what defines the rules for the bucket (who can access, who can read objects stored in it and so on). 

To create the policy we take advantage of an AWS tool called [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html).

- on the policy generator page one of the first options is to choose the type of policy. Select 'S3 Bucket Policy';
- on step 2 (Add statements) the 'Effect' option is 'Allow';
- the 'Principal' is the ARN (Amazon Resource Name) of our IAM user created previously. We can find it in the 'Users' section from IAM page, where we have our users listed. Just copy the ARN and paste in the 'Principal' field;
- in the 'Actions' field find an option called 'PutObject' and select it;
- the last field is the 'Amazon Resource Name (ARN)', and this is our bucket ARN. To find it go to Amazon S3 service page > Buckets and click in the bucket we created before. In the 'Properties' section find the ARN value. Copy it and paste in the 'Amazon Resource Name' field;
- now we can click in 'Add Statement' and then 'Generate Policy'.   

Basically we created a policy that allows the Principal (our IAM user) to put objects in our S3 bucket. Now we have to attach this policy to the bucket.

On the 'Permissions' section, in your new bucket page, find a big field called 'Bucket policy'. You can 'Edit' this field, paste the new policy and then 'Save changes'.

## A little about Django 'collectstatic' command

Before setting up our application, it's nice to understand a little about how Django manages static files. 

Django has a built-in application called ['staticfiles'](https://docs.djangoproject.com/en/5.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles), located in django.contrib. This app is responsible for collecting static files from each of your applications into a single location defined in your [STATIC_ROOT](https://docs.djangoproject.com/en/5.0/ref/settings/#std-setting-STATIC_ROOT) setting (settings.py).

So, when you run the command ```python manage.py collectstatic```, all the static files located in 'static' subdirectories inside your apps will be collected into STATIC_ROOT. The files are collected from 'static' subdirectories because of the default STATICFILES_FINDERS setting. You can [check it](https://docs.djangoproject.com/en/5.0/ref/settings/#staticfiles-finders) if necessary.

To give a quick example, let's suppose you have this configuration in your settings.py and the following folder structure:

```python
settings.py

STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
```
BASE_DIR is a variable that points to the root directory of your application.

```
├── 📂 root_directory
|   ├── manage.py
|   ├── .env
|   ├── 📂 project
|   |   ├── settings.py
|   |   ├── 📂 base                 # application
|   |   |   ├── 📂 static           # static directory
|   |   |   |   ├── 📂base
|   |   |   |   |   ├── 📂css
|   |   |   |   |   |   ├── base.css
```

After running the 'collectstatic' command a 'staticfiles' directory will be created in the root_directory folder and it would look like this:

```
├── 📂 root_directory
...
|   ├── 📂 staticfiles          # new directory
|   |   ├── 📂base              # folder collected
|   |   |   ├── 📂css
|   |   |   |   ├── base.css
```

# Setting up our project

Now we can configure the project to send our static files to our S3 bucket. A '.env' file in the root of the project will be necessary, and you have to fill it with the variables that we talked about previously. The values for these environment variables were also talked about in the previous steps of this tutorial, and you have to use them here.

```
.env

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_STORAGE_BUCKET_NAME=
```

We also have to add some settings to our settings.py:

```python
settings.py

STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

if AWS_ACCESS_KEY_ID:
    AWS_SECRET_ACCESS_KEY = config('AWS_SECRET_ACCESS_KEY')         # python-decouple
    AWS_STORAGE_BUCKET_NAME = config('AWS_STORAGE_BUCKET_NAME')     # python-decouple

    STORAGES = {
        "default": {
            "BACKEND": "storages.backends.s3.S3Storage",            # django-storages[s3]
        },
        "staticfiles": {
            "BACKEND": "storages.backends.s3.S3Storage",            # django-storages[s3]
            "OPTIONS": {
                'bucket_name': AWS_STORAGE_BUCKET_NAME,
                'object_parameters': {'CacheControl': 'max-age=86400'},
                'file_overwrite': False,
            }
        },
    }
```
The 'default' key in the STORAGES setting is to deal with media files (uploaded by a user, for example).

With these settings added we are basically telling Django that, if we have a value set for AWS_ACCESS_KEY_ID (in our .env), read the other two environment variables set and use our custom STORAGES setting to deal with our files. And now, by running the command ```python manage.py collectstatic```, the static files will be sent to the bucket in AWS.

So, if you don't want to send the static files to the new S3 bucket, just remove the value from the AWS_ACCESS_KEY_ID setting in the .env file. By doing this, django will not use the custom STORAGES setting, it will use its [default implementation](https://docs.djangoproject.com/en/5.0/ref/settings/#storages) and send the static files to the 'staticfiles' directory (since we defined it in STATIC_ROOT) that we talked about before.

To finish, set the three AWS variables from .env file in your production environment to have your files collected into the S3 bucket when you run 'collectstatic' command during deploy.

## References

- https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html?icmpid=docs_iam_console
- https://awspolicygen.s3.amazonaws.com/policygen.html
- https://docs.djangoproject.com/en/5.0/howto/static-files/
- https://docs.djangoproject.com/en/5.0/ref/settings/
- https://docs.djangoproject.com/en/5.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles
- https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html
