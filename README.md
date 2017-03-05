Fabric AWS CloudFormation
====

A Python library that generates [Fabric](http://www.fabfile.org) tasks to manipulate the stack of AWS CloudFormation.

**This repository in sandbox now.**
**Do not use in your product development.**

# Setup

## Install

Install 'fabricawscf'([and dependencies](./setup.py)) via pip.

```bash
pip install git+https://github.com/crossroad0201/fabric-aws-cloudformation.git
```

## Uninstall

```bach
pip uninstall fabricawscfn
```

# Usage

## Preparation

* Create S3 bucket for store CloudFormation templates.

## Import 'fabricawscfn' in your 'fabfile.py'.

```python
from fabricawscfn import *
```

## Define stacks and generate Fabric tasks.

```python
StackGroup('my-cfn-templates', 'example', 'templates')\
  .define_stack('foo', 'example-foo', 'foo.yaml')\
  .define_stack('bar', 'example-bar', 'bar.yaml', Tags=[{'Key':'example', 'Value':'EXAMPLE'}])\
  .generate_task(globals())
```

### Create StackGroup

Instantiate 'StackGroup'.

* Parameters.
  * 'templates_s3_bucket' - Prepared S3 bucket name.
  * 'templates_s3_prefix' - Prefix(Folder) name in Prepared S3 bucket. CloudFormation templates store in.
  * 'templates_local_dir'(OPTIONAL) - Local dir(relative path) where CloudFormation template(s) stored.

* 'templates_s3_bucket' and 'templates_s3_refix' can contains placeholder(Like ''%(environment)s'). Replace by Fabric env.

### Define Stack

Define Stack using 'StackGroup#define_task()'.

* Parameters.
  * 'alias' - Alias(Short name) of Stack. This name is using task parameter.
  * 'stack_name' - CloudFormation Stack name.
  * 'template_path' - Template file path.(Relative path from 'templates_local_dir')
  * ''**kwargs' - Additional arguments for Create/Update stack. See [Boto3 reference](https://boto3.readthedocs.io/en/latest/reference/services/cloudformation.html#CloudFormation.Client.create_stack).

* 'templates_s3_bucket' and 'templates_s3_refix' can contains placeholder(Like ''%(environment)s'). Replace by Fabric env.

### Generate Task

Generate Fabric tasks using 'StackGroup#generate_task()'.

* Parameters.
  * 'namespace' - Generated tasks added to this namespace. Normaly specify 'globals()'.

## Finish

You can check generated tasks run 'fab -l' comand.

```bash
$ fab -l
Available commands:

    create_bar         create stack bar.
    create_foo         create stack foo.
    delete_bar         delete stack bar.
    delete_foo         delete stack foo.
    desc_stack         Describe existing stack.
    ls_exports         List exports.
    ls_resources       List existing stack resources.
    ls_stacks          List stacks.
    params             Set parameters. (Applies to all tasks)
    sync_templates     Synchronize templates local dir to S3 bucket.
    update_bar         update stack bar.
    update_foo         update stack foo.
    validate_template  Validate template on local dir.
```

# Example

See [Example fabfile.py](./example/fabfile.py).

## Basic Tasks.

### 'ls_stacks'

Show stacks list.

```bash
$ fab ls_stacks
Stacks:
+------------+----------------------+-----------------+----------------------------------+-------------+-------------+
| StackAlias | StackName            |      Status     |           CreatedTime            | UpdatedTime | Description |
+------------+----------------------+-----------------+----------------------------------+-------------+-------------+
| foo        | fabricawscfn-dev-foo | CREATE_COMPLETE | 2017-03-05 04:35:12.823000+00:00 |      -      | Foo bucket. |
| bar        | fabricawscfn-dev-bar |   Not created   |                -                 |      -      | -           |
+------------+----------------------+-----------------+----------------------------------+-------------+-------------+
```

### 'desc_stack:xxx'

Show stack detail(Parameters, Outputs, Events).

```bash
$ fab desc_stack:foo
Stack:
+----------------------+-----------------+----------------------------------+-------------+-------------+
| StackName            |      Status     |           CreatedTime            | UpdatedTime | Description |
+----------------------+-----------------+----------------------------------+-------------+-------------+
| fabricawscfn-dev-foo | CREATE_COMPLETE | 2017-03-05 04:35:12.823000+00:00 |     None    | Foo bucket. |
+----------------------+-----------------+----------------------------------+-------------+-------------+
Parameters:
+---------+--------+
| Key     | Value  |
+---------+--------+
| Param4  | PARAM4 |
| Param3  | PARAM3 |
| Param2  | PARAM2 |
| Param1  | PARAM1 |
| EnvName | dev    |
+---------+--------+
Outputs:
+--------+-----------------+-------------+
| Key    | Value           | Description |
+--------+-----------------+-------------+
| Bucket | sandbox-dev-foo | Foo bucket. |
+--------+-----------------+-------------+
Events(last 20):
+----------------------------------+--------------------+----------------------------+----------------------+-----------------------------+
| Timestamp                        |       Status       | Type                       | LogicalID            | StatusReason                |
+----------------------------------+--------------------+----------------------------+----------------------+-----------------------------+
| 2017-03-05 04:35:55.694000+00:00 |  CREATE_COMPLETE   | AWS::CloudFormation::Stack | fabricawscfn-dev-foo | None                        |
| 2017-03-05 04:35:53.009000+00:00 |  CREATE_COMPLETE   | AWS::S3::Bucket            | Bucket               | None                        |
| 2017-03-05 04:35:32.308000+00:00 | CREATE_IN_PROGRESS | AWS::S3::Bucket            | Bucket               | Resource creation Initiated |
| 2017-03-05 04:35:31.102000+00:00 | CREATE_IN_PROGRESS | AWS::S3::Bucket            | Bucket               | None                        |
| 2017-03-05 04:35:12.823000+00:00 | CREATE_IN_PROGRESS | AWS::CloudFormation::Stack | fabricawscfn-dev-foo | User Initiated              |
+----------------------------------+--------------------+----------------------------+----------------------+-----------------------------+
```

### 'validate_template'

Validate CloudFormation template.

```bash
$ fab validate_template:bar
Validating template templates/subdir/bar.yaml...
[localhost] local: aws cloudformation validate-template --template-body file://templates/subdir/bar.yaml --output table
--------------------------------------------------------------------
|                         ValidateTemplate                         |
+--------------------------------+---------------------------------+
|  Description                   |  Bar bucket.                    |
+--------------------------------+---------------------------------+
||                           Parameters                           ||
|+--------------+----------------------+---------+----------------+|
|| DefaultValue |     Description      | NoEcho  | ParameterKey   ||
|+--------------+----------------------+---------+----------------+|
||  dev         |  Environmanet name.  |  False  |  EnvName       ||
|+--------------+----------------------+---------+----------------+|
```

### 'sync_templates'

Upload CloudFormation templates to S3 bucket.

```bash
$ fab sync_templates
Synchronizing templates local templates to s3://crossroad0201-fabricawscfn/example/dev...
[localhost] local: aws s3 sync templates s3://crossroad0201-fabricawscfn/example/dev --delete --include "*.yaml"
upload: templates\foo.yaml to s3://crossroad0201-fabricawscfn/example/dev/foo.yaml
```

### 'crate_xxx'

Create new stack.

You can specify Stack parameter(s) via task parameter.(Like this '$ fab create_xxx:Param1=PARAM1,Param2=PARAM2')
If parameters are not specified by task parameter, prompt will be displayed and input will be prompted.

```bash
$ fab create_bar
Creating stack...
  Stack Name: fabricawscfn-dev-bar
  Template  : https://s3.amazonaws.com/crossroad0201-fabricawscfn/example/dev/subdir/bar.yaml
  Parameters: [{'ParameterValue': 'dev', 'ParameterKey': 'EnvName'}]
Waiting for complete...
Finish.
```

## Optional Tasks

### 'params'

Specify Stack parameters bulkly.

```bash
$ fab params:Param1=PARAM1,Param2=PARAM2 create_xxxx create_yyyy
```

# Links

* [Fabric](http://www.fabfile.org)
* [bot3 CloudFormation](https://boto3.readthedocs.io/en/latest/reference/services/cloudformation.html)

* [一歩すすんだ Fabric のタスク定義のしかた](https://nulab-inc.com/ja/blog/backlog/fabric-advanced/)
* [Python製デプロイツール Fabricを初めて使う際に役立つTip](http://dekokun.github.io/posts/2013-04-07.html)

* [GithubにPythonのライブラリをあげてpipでインストールする](http://blog.junion.org/pip-github-test/)
* [Pythonによる CLI ツールの実装と配布](http://developer.wonderpla.net/entry/blog/engineer/python_cli_tool_implementation_and_distribution/)
