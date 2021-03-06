[![Build Status](https://travis-ci.org/mixi-inc/datadog-aws-ec2-counter.svg?branch=master)](https://travis-ci.org/mixi-inc/datadog-aws-ec2-counter)

# datadog-aws-ec2-counter
This is an Agent Check for [Datadog](https://www.datadoghq.com/) for obtaining On-Demand and Reserved Instances count of AWS EC2.
The custom metrics that can be obtained with this Agent Check are as follows.

- Count of active EC2 On-Demand Instances and [footprint](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ri-modification-instancemove.html)
- Count of active EC2 Reserved Instances and footprint
- Count of unused EC2 Reserved Instances and footprint
- Total count of active EC2 Instances and footprint

By using these custom metrics, you can refer to contracts of Reserved Instances or discover unused Reserved Instances.

This information can also be confirmed in the EC2 report of the AWS console, but by using this Agent Check it becomes possible to grasp in detail the usage situation in real time every minute.

![example](https://raw.githubusercontent.com/mounemoi/datadog-aws-ec2-counter/images/example.png "example")

## Metrics

The list of metrics acquired with this Agent Check is as follows.

| Metrics | Description |
|-|-|
| aws_ec2_count.ondemand.count | Count of active EC2 On-Demand Instances |
| aws_ec2_count.ondemand.footprint | Footprint of active EC2 On-Demand Instances |
| aws_ec2_count.reserved.count | Count of active EC2 Reserved Instances |
| aws_ec2_count.reserved.footprint | Footprint of active EC2 Reserved Instances |
| aws_ec2_count.reserved_unused.count | Count of unused EC2 Reserved Instances |
| aws_ec2_count.reserved_unused.footprint | Footprint of unused EC2 Reserved Instances |
| aws_ec2_count.running.count | Total count of active EC2 Instances |
| aws_ec2_count.running.footprint | All footprint of active EC2 Instances |

Each metric has the following tags, from which you can determine its Availability Zone and Instance Type.

| Tag | Description |
|-|-|
| ac-az | Availability Zone (or 'region' in the case of Reserved Instances in a region) |
| ac-family | Instance Family |
| ac-type | Instance Type |

## Prepare

Prepare the following EC2 Instance.

- Server on which [Datadog Agent](http://docs.datadoghq.com/guides/basic_agent_usage/) is installed
- Grant `ec2:DescribeInstances` authority in IAM Role

Install this Agent Check on this server.

## How to setup

This section describes how to install this Agent Check on Datadog Agent installed on CentOS. You may need to adjust depending on the installation environment.

### 1. Install AWS SDK

Install [AWS SDK for Python](https://aws.amazon.com/sdk-for-python/). It's required for this Agent Check.

```bash
$ sudo /opt/datadog-agent/embedded/bin/pip install boto3
```

### 2. Install this Agent Check
Place `./checks.d/aws_ec2_count.py` from this repository into `/etc/dd-agent/checks.d/`.

```bash
$ sudo cp ./checks.d/aws_ec2_count.py /etc/dd-agent/checks.d/
```

### 3. Place Agent Check configuration file
Create `/etc/dd-agent/conf.d/aws_ec2_count.yaml` by referring to `./conf.d/aws_ec2_count.yaml.example` in this repository.

```yaml:aws_ec2_count.yaml
init_config:
    min_collection_interval: 60

instances:
    - region: 'ap-northeast-1'
```

- `min_collection_interval` specifies the request interval (in seconds)
- `region` specifies the region to be acquired. Several regions can be speficied in `instances` as an array.

If the acquisition target is a Tokyo region (ap-northeast-1), you can use this `aws_ec2_count.yaml.example` as it is.

```bash
$ sudo cp conf.d/aws_ec2_count.yaml.example /etc/dd-agent/conf.d/aws_ec2_count.yaml
```

### 4. Restart Datadog Agent
Finally restart Datadog Agent.

```bash
$ sudo /etc/init.d/datadog-agent restart
```

Your custom metrics should now be sent to Datadog.

## Restrictions
This Agent Check has the following restrictions.

- The count of On-Demand Instances is calculated as the difference between the count of active instances and the count of valid Reserved Instances.
    - Because of that, it may not exactly match the invoiced amount.
    - In addition, due to changes in AWS specifications in the future, the applicable conditions of Reserved Instances may change.
    - Please use this Agent Check's data as a reference only. For exact data always refer to AWS.
- Regional Reserved Instances are calculated with applied discounts under the following conditions.
    - Priority is given to the same Instance Type.
    - The surplus is applied to those of the same Instance Family, starting with the smallest Instance Size.
        - This makes it possible to minimize the count of On-Demand Instances.
- There are times when it is not possible to correctly acquire the count of Reserved Instances depending on the timing Reserved Instances are changed.
- Only the following instances are supported.
    - Platform is `Linux/UNIX`.
    - Tenancy is `default`.
    - Scheduled Reserved Instances are not supported.

# License
This software is released under the [MIT License](http://opensource.org/licenses/MIT), see LICENSE.

