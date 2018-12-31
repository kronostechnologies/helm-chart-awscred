# helm-chart-awscred
This chart is intended to install [upmc-enterprises/registry-creds](https://github.com/upmc-enterprises/registry-creds) when you have your ECR registry in another account than your kubernetes cluster. If this is not your case, you may modify some part of this code to get your case working.

## Build
These commands will create `awscred` folder and a `.tgz` file.
```
git clone git@github.com:kronostechnologies/helm-chart-awscred.git awscred
helm package awscred
```

## Install
Before installing this chart, the kubernetes cluster needs some initial aws role properly configured.

### Roles
The AWS account containing the ECR needs a role. For the sake of the example, we will call the account `1234567890` and the role `EcrReadOnly`. The policy of this role needs to be like below.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "AWS": [
          "arn:aws:iam::2234567890:root"
        ],
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
```
In this example, the account `2234567890` will be able to access the role `EcrReadOnly` located in account `1234567890`. If you have more account, simply add them to the `AWS` array.

### Node Role Policy
You kubernetes node need at least this policy on their role.
```
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": [
    "arn:aws:iam::1234567890:role/EcrReadOnly"
  ]
}
```
This allow the kubernetes node to assume the role `EcrReadOnly` in account `1234567890`.

### Chart installation
Install this chart with the command below. Make sure to replace awsAccount= and awsRole= with the proper account number and role name. In our example, it would be a command like this.

```
helm install --namespace kube-system --name awscred --set-string awsAccount=1234567890 --set-string awsRole=arn:aws:iam::1234567890:role/EcrReadOnly https://github.com/kronostechnologies/helm-chart-awscred/releases/download/0.2.0/awscred-0.2.0.tgz
```

If you want to use the local build version of this chart.
```
helm install --namespace kube-system --name awscred --set-string awsAccount=1234567890 --set-string awsRole=arn:aws:iam::1234567890:role/EcrReadOnly awscred
```

## Release
To make a release, tag the specific commit you need, upgrade the version of the chart (as well as examples in this documentation refering to the latest version), push everything, create a new release in github and add the .tgz file in the version.

```
# replace $oldversion with $newversion both in documentation and Chart.yaml
git add .
git commit -m "Version $newversion"
git tag -a $newversion -m "$newversion"
git push --tags
helm package .
# go to github, create the release and upload the built package into the release.
```
