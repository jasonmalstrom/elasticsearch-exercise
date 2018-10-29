# elasticsearch-exercise
An  exercise to demonstrate standing up Elasasticsearch in AWS

Will be documenting as I go along, so this may seem a bit like a train of thought.

I ran these playbooks off of a Xenial Ubuntu box, using Ansible 2.7, but tried to stay within Ansible 2.4 for compatibility. I found I had to run the following commands to make all the local Ansible modules work correctly.

For attempted simplicity, this install Elasticsearch on an Amazon Linux EC2 Instance

## Setup

The repository can downloaded via
```
git clone https://github.com/jasonmalstrom/elasticsearch-exercise.git
```

Ansible will come configured to use a vault password file outside of the git project.
From inside the project, run:
```
echo "9gT&xgp[xK#R<9v" | cat > ../.ansible-vault
```

Setup some environment variables that match up to AWS keys that will be used
```
export AWS_KEY=example2018
export AWS_PEM=~/.ssh/example.pem
```

## Method 1

Setup AWS infra
```
ansible-playbook site.yml -t aws --extra-vars "my_key=$AWS_KEY" -vvv
```

Grab the new EC2 Instance from the output or AWS console
```
export ES_IP=<Instance IP address>
ansible-playbook -i $ES_IP, site.yml -b -u ec2-user --key-file=$AWS_PEM -vvv
```

You can verify Elasticsearch via Ansible which will run a https request from you machine to the Elasticsearch instance
```
ansible-playbook site.yml -t test -vvv --extra-vars "ip_address=$ES_IP"
```

To tear down resources created, you can run:
```
ansible-playbook cleanup.yml
```

## Method 2

In preparation to expand into a cluster, I decided that it would be better to run this in a more production friendly method within an autoscaling group. The following only sets up a single node instance, but would be much easier to convert to a multi-node install.

You will need to have packer from hashicorp installed locally
```
packer build elasticsearch.json
```

when packer reports back the AMI, use it in the environment variable
```
export ES_AMI=<packer created ami>
ansible-playbook cluster.yml --extra-vars "my_key=$AWS_KEY, image_id=$ES_AMI" -vvv
```

Again, you can clean up your resources with this playbook, although the AMI will need to be manually deleted
```
ansible-playbook cleanup.yml
```

## Design Considerations

I decided to base most of the exercise on Ansible, since I find that it's easiest for it to play nice in any environment and doesn't require as much knowledge of where it's going in. Plus, it tends to be a swiss army and can reasonably do many types of tasks. This allowed me to utilize Ansible how one might normally utilize jobs Jenkins, although more crude and less complete.  

While I originally considered using all the Ansible AWS modules, I ended up decided that I would use it as a launcher for CloudFormation, as that would allow for easy and neat build up and tear downs in testing. This was indeed useful, although using Ansible might have been quicker to do the actual coding with.

In my job, I most recently launched Elasticsearch using the ansible modules they provide, but knew that would be outside the spirit of this exercise, so I wrote mine from scratch.

I originally was going to secure the Elasticsearch instance with Search Guard, but that ended up requiring a lot of configuration beyond the scope of this project, so I decided that just using NGINX would be the simpler solution. Especially as I no longer had to have Elasticsearch talk directly to the outside. In a production environment Search Guard would have had more value, as it also encrypts node traffic.

I did like how I was able to leverage Ansible as also a testing platform to verify the install worked, thus being able to use a single common secret management.

If I was do start fresh, I would have started with autoscaling packer made images.

##  Resources

https://docs.ansible.com/
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
https://www.elastic.co/guide/en/elastic-stack/current/index.html
https://docs.search-guard.com/latest/search-guard-community-edition
https://www.elastic.co/guide/en/elasticsearch/reference/master/max-number-of-threads.html
https://hostpresto.com/community/tutorials/install-and-secure-nginx-on-centos-7/
https://stackoverflow.com/questions/38423219/ansible-this-module-requires-the-passlib-python-library
https://www.packer.io/docs/builders/amazon-ebs.html
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-autoscaling.html
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-as-launchconfig.html

## Feedback

I did have fun doing this exercise and it was generally enjoyable. It did seem to take a little longer than expected, but I had a bit of trying to get my home machine up to date, especially with various python libraries. Also going down the wrong path with Search Guard ate a fair amount of time. I did encounter a number of weird gotchas along the way, which debugging those can be a time sink. I think it would have help to have tackled it in one sitting at a time that I was fresh, instead of over the course of several nights near bed time.
