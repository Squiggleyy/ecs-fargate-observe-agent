	•	Define a task definition with 2 containers:
	1.	Your test container (e.g., Alpine running a loop)
	2.	Observe Agent sidecar container
	•	Deploy a task based on that definition
	•	ECS will spin up both containers together on the same Fargate node
	•	They’ll share a network (so your app can send traces to localhost:4317)
	•	If the Observe Agent is marked as essential: false, your app keeps running even if the agent crashes


aws configure
aws ecs register-task-definition --cli-input-json file://observe-test-task.json --debug

aws ecs run-task \
  --cluster your-cluster-name \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxxxxx],securityGroups=[sg-xxxxxxxx],assignPublicIp=ENABLED}" \
  --task-definition observe-test-task:3

aws ec2 describe-vpcs --filters Name=isDefault,Values=true
aws ec2 describe-subnets --filters Name=vpc-id,Values=<vpc-id>

aws ec2 describe-security-groups --filters Name=vpc-id,Values=<vpc-id>

aws ec2 create-security-group \
  --group-name fargate-test-sg \
  --description "Allow outbound traffic" \
  --vpc-id <vpc-id>

aws ec2 authorize-security-group-egress \
  --group-id <sg-id> \
  --protocol -1 \
  --port all \
  --cidr 0.0.0.0/0