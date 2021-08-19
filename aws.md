Amazon Web Services (AWS) Integration
========================

Ops can integrate with your existing Amazon Web Services (AWS) account. You can use the Ops CLI to create and upload an image in your AWS account.
Once, you have uploaded the image, you can also create an instance with a particular image using CLI.

We currently automate installing a vmimport role
(https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-troubleshooting.html).
It needs to be tied to the bucket you use and uses an unique name. If
you are having trouble you might wish to see if you have an existing
vmimport role defined already.

## Instance type Support

Nanos supports running on many of the AWS instance types including
normal instance types like t2 and the newer Nitro based instances such as t3, c5, etc.

We currently don't have support for ARM based instances.

## Pre-requisites

1. Ensure your ~/.aws/credentials file is setup correctly - you can use
   the 'aws ec2' cli tools to verify this.

4. Create a bucket in S3 storage for ami creation.


## Image Operations
### Create Image

If you have already created an Ops project, you can update your existing `config.json`.
You need to add [CloudConfig](configuration.md#cloudconfig) which mentions specific details like `ProjectID`, `BucketName`.

```json
{
    "CloudConfig" :{
        "ProjectID" :"prod-1000",
        "Zone": "us-west-1",
        "BucketName":"my-s3-bucket"
    }
}
```

Once, you have updated `config.json` you can create an image in AWS with the following command:

For creating an image using a particular package, you need to provide the package name to `ops image create` command with `-p` option.

```sh
$ ops image create -t aws -c config.json -p node_v14.2.0 -a ex.js
```

### List Images

You can list existing images on AWS with `ops image list`.

```sh
$ ops image list -t aws -z us-west-1

+----------------+-----------+--------------------------+
|      NAME      |  STATUS   |         CREATED          |
+----------------+-----------+--------------------------+
| nanos-webg-ata | available | 2019-07-16T05:56:06.000Z |
+----------------+-----------+--------------------------+
| will-00        | available | 2019-08-07T20:21:19.000Z |
+----------------+-----------+--------------------------+
| nanos-webg     | available | 2019-07-16T05:01:44.000Z |
+----------------+-----------+--------------------------+
| node-image     | available | 2019-08-15T22:31:03.000Z |
+----------------+-----------+--------------------------+
```

### Delete Image

`ops image delete -i <imagename>` can be used to delete an image from AWS.

```
$ ops delete image -i nanos-main-image
```

## Instance Operations
### Create Instance

After the successful creation of an image in AWS, we can create an instance from an existing image.

```
$ ops instance create <image_name>
```

Alternatively, you can pass project-id and zone with cli options.

```sh
$ ops instance create <image_name> -p prod-1000 -z us-west-2
```

You can also pass config, if you have mentioned project-id and zone in project's config.json.
```
$ ops instance create <image_name> -c config.json
```

You can provide list of ports to be exposed on aws instance via config and command line.

CLI example
``` sh
$ ops instance create <image_name> -p prod-1000 -z us-west-2 --port 80 --port 443
```

Sample config

```json
{
    "CloudConfig" :{
        "Platform" :"aws",
        "ProjectID" :"prod-1000",
        "Zone": "us-west-1",
        "BucketName":"my-s3-bucket"
    },
    "RunConfig": {
        "Ports" : [80, 443]
    }
}
```

If you need to specify a specific zone when creating the instance use
the [Subnet config var](https://nanovms.gitbook.io/ops/configuration#cloudconfig.subnet).

This is necessary for certain operations such as when you want to mount
a volume to an instance.

#### VPC and Security Group

By default, ops uses the first VPC found in aws and creates a security group per instance.

You can select a different VPC or use a existing security group using the configuration file. The keys to set are `RunConfig.VPC` and `RunConfig.SecurityGroup`.
```json
{
    "RunConfig":{
        "VPC": "vpc-name",
        "SecurityGroup": "sg-name"
    }
}
```

### List Instances

You can list instance on AWS using `ops instance list` command.

```sh
$ ops instance list -p prod-1033 -t aws -z us-west-2

$ ops instance list
+------+----+--------+---------+-------------+------------+
| NAME | ID | STATUS | CREATED | PRIVATE IPS | PUBLIC IPS |
+------+----+--------+---------+-------------+------------+
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance list -p prod-1000 -z us-west1-b
```

### Get Logs for Instance

You can get logs from serial console of a particular instance using `ops instance logs` command.

```sh
$ ops instance logs <instance_name>
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance logs -t aws -p prod-1033 -z us-west-2 i-08815dc4b29b44294
```

### Delete Instance

`ops instance delete` command can be used to delete instance on AWS.

```
$ ops instance delete my-instance-running
```

Alternatively you can pass project-id and zone with cli options.
```sh
$ ops instance delete -p prod-1000 -z us-west1-b my-instance-running
```
