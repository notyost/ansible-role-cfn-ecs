# ansible-role-cfn-ecs

## Top level attributes

| Variable        | Required           | Default  | Description |
| ------------- |:-------------:| -----:| -------------------------------------------------------------------------:|
| ecs_host_keypair_name | yes | | the keypair name used to log into the host
| ecs_subnet | yes || list of comma separated subnets (1 or more) into which the host will be deployed |
| ecs_region | yes | | region into which to deploy the cluster |
| ecs_container_name | yes | |  name of to use for the container |
| ecs_zone_name | yes | | The name of the hosted zone where the domain name lives |
| ecs_domain_name | yes | | The domain name used by the load balancer fronting the cluster |
| ecs_host_vpc | yes | | The VPC that the ECS cluster will live in |
| ecs_stack_name | yes | | The CloudFormation stack name |
| ecs_containers | yes | | A list of container definitions utilized in the cluster. |
| ecs_elb | yes | | An ELB definition used to front the cluster. |
| ecs_elb_access_cidr | yes | | The CIDR range of users of the ELB |
| ecs_container_port | yes | | Port to run the container on |
| ecs_host_ami | yes | | AMI for the host instance |
| ecs_tags | yes | | Set of tags to use with the ECS stack |
| ecs_creation_timeout | no | PT45M | Length of time before stack creation times out |
| ecs_update_timeout | no | PT45M | Length of time before stack update times out |
| ecs_asg_desired_capacity | no | 2 | Desired capacity for the ASG used for ECS |
| ecs_asg_max_capacity | no | 2 | Max capacity for the ASG used by ECS |
| ecs_asg_min_capacity | no | 1 | Min capacity for the ASG used by ECS |
| ecs_key_name | no | <strong>ecs_stack_name</strong> | |
| ecs_max_cluster_size | no | 1 | |
| ecs_instance_type | no | t2.micro | |
| ecs_credstash_kms_key_arn      | no |  | If an application on the cluster requires a secret from credstash, provide the pertinent KMS key  |
| ecs_credstash_table_arn | no | | If an application on the cluster requires a secret from credstash, provide the pertinent dynamodb table |
| ecs_build_dir | no | build | Directory to be used to store temporary build artifacts |
| ecs_template_dir | no | <strong>ecs_build_dir</strong> | Directory be used to store generated templates |

### Attributes of ecs_containers

| Variable        | Required           | Default  | Description |
| ------------- |:-------------:| -----:| -------------------------------------------------------------------------:|
| cpu | yes | none | |
| image | yes | none | docker image to use for the container |
| port_mappings | yes | none | Mapping from host to container port. Format is "host_port" : "container_port" |  
| memory | yes | none | amount of memory to allocation to the container |

### Attributes of ecs_elb

| Variable        | Required           | Default  | Description |
| ------------- |:-------------:| -----:| -------------------------------------------------------------------------:|
| health_check | yes | | Define the ELB health check attributes |
| listeners | yes | | Define the listers that will be used by the ELB |

#### Attributes of ecs_elb.health_check

| Variable        | Required           | Default  | Description |
| ------------- |:-------------:| -----:| -------------------------------------------------------------------------:|
| target | yes | | Path on the backing ECS instance to perform the health check against. Format is <strong>PROTOCOL:PORT/PATH</strong> |
| healthy | yes | | Number of checks necessary to consider instance healthy | 
| unhealthy | yes | | Number of checks necessary to consider instance unhealhy |
| interval | yes | | Amount of time between health checks |
| timeout | yes | | Length of time to wait for response from instance |

#### Attributes of ecs_elb.listeners

| Variable        | Required           | Default  | Description |
| ------------- |:-------------:| -----:| -------------------------------------------------------------------------:|
| elb_port | yes | | Port on the ELB that will receive traffic |
| instance_port | yes | | Port on the backing ECS instance that the ELB will communication with |
| protocol | yes | | Protocol used to communicate with the ELB |
| certificate | conditional | | Required if protocol is https. This specifies the ARN of the SSL certificate to be used by the ELB |


### Example playbook

```
- hosts: localhost
  connection: local
  gather_facts: False
  vars:
    hosted_zone: "some.zone"
    ecs_zone_name: "{{ hosted_zone }}."
    ecs_domain_name: "host.{{ hosted_zone }}"
    ecs_host_vpc: "vpc-123456789"
    ecs_stack_name: "test-ecs"
    ecs_credstash_table_arn: "arn:aws:dynamodb:us-west-2:123456789012:table/some-dynamodb-table"
    ecs_credstash_kms_key_arn: "arn:aws:kms:us-west-2:123456789012:key/some-kms-key-id"
    ecs_containers:
      - cpu: 100
        image: "1234567890.dkr.ecr.us-west-2.amazonaws.com/some_image:latest"
        port_mappings:
          "80" : "8081"
        memory: 300
        elb: "{{ ecs_elb }}"
    ecs_elb:
      listeners:
        - elb_port: 443
          instance_port: 80
          protocol: HTTPS
          certificate: "arn:aws:iam::1234567890:server-certificate/some-cert"
      health_check:
        target: "HTTP:8081/"
        healthy: 3
        unhealthy: 3
        interval: 20
        timeout: 5
    ecs_container_name: some_container
    ecs_elb_access_cidr: 1.0.0.0/32
    ecs_use_elastic_ips: "true"
    ecs_container_port: 80
    ecs_host_ami: "ami-1234567890"
    ecs_subnet: "subnet-1234567890"
    ecs_host_keypair_name: "some-keypair"
    ecs_tags:
        "Name" : "some-instance-tag"
  roles:
    - role: ecs
```
