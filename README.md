
# Web-calculator

web-calculator is a web service that allows users to send arithmetic expressions via HTTP and receive their results

## Functionality

- Supports addition, subtraction, multiplication, division operations, and expressions in parentheses.
- Expressions can be entered with or without spaces between numbers and operands.
- The calculator accepts positive integers as input.

## Requirements

- Python (depends on the linux distribution)
   ```bash
   sudo apt-get update
   sudo apt-get install python3.8
   ```
- Pip
   ```bash
   curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
   python get-pip.py --user
   ```
- Ansible (depends on the linux distribution)
   ```bash
   sudo apt-get install ansible
   ```

## Installation

1. Clone the repository

   ```bash
   git clone https://github.com/eitanas1/web-calculator
   cd web-calculator
   ```

2. Configure k8s control-plane's IP and k8s nodes's IP in ansible/inventory.ini file and test the connection

   ```bash
   ansible all -i ansible/inventory.ini -m ping
   ```

3. Setup k8s cluster using Ansible playbooks
   ```bash
   ansible-playbook -i ansible/inventory.ini ansible/web-calculator-playbook.yml
   ```

4. Run the server from the helm chart
   
   ``` bash
   helm dep up helm
   helm install web-calculator helm
   ```

## Environment Variables (using Helm chart)

| Variable | Description | Default Value |
|----------|----------------|----------------|
| service.type | Service type | ClusterIP |
| service.port | Service port | 8080 |
| env.logLevel | Application log level | INFO |
| user.username | Dummy application username (only for k8s secret example) | dummy_username |
| user.password | Dummy application user password (only for k8s secret example) | dummy_password |

## Security Measurements Implemented
During the CI process security measurements had implemented using those tools:
1. OWASP Dependency-Check - Software Composition Analysis (SCA) tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies.
2. Anchore Grype - vulnerability scanner for container images and file systems.

## Monitoring Tools
In order to monitor the app in k8s cluster, Prometheus and Grafana can be installed:
   ### Prometheus:
   ``` bash
   kubectl create namespace monitoring
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install prometheus prometheus-community/prometheus --namespace monitoring
   ```
   ### Grafana:
   ``` bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo update
   helm install grafana grafana/grafana --namespace monitoring
   ```

## Challenges faced and solutions applied
* The list of example demo-apps (https://github.com/devopsdemoapps) doesn't contain any apps that are not dockerized. Some of the example demo-apps lack documentation and don't compile or function as expected. To solve it I was looking for other demo-apps that are not dockerized, include sophisticated documentation and can compile and run as expected.
* Providing Jenkins instance on AWS EC2 under the free tier program results in an instance with less resources needed for using different security measurement tools. I was configuring and trying different solutions but some of them couldn't run in this environment. Eventually I picked up OWASP Dependency-Check which is functioning but consuming a lot of data (around 700mb) and Anchore Grype for scanning the docker image for vulnerabilities.
* Ansible is a tool that I never used before so I needed to learn the basics from scratch. Honestly, I found it pretty interesting and efficient.
* The web-calculator app doesn’t require any variables or secrets. In order to demonstrate variables and secrets implementation in the Helm chart, I added a dummy variable and secrets to the Helm chart.

## Improvement points
* Jenkins - for cost management purposes, the Jenkins controller (master) is also used as a build executor. For a better stability and performance one or more Jenkins nodes should be added as build executors.
* Application Scalability - autoscaling needed to be configured in order to handle different workloads.
* Secrets management tools - k8s external secrets operator should be added in order to pull secrets dynamically from secrets management tools like HashiCorp Vault, AWS Secret Management etc.
* Test reports - the results of unit tests and security tests should be published as an html report or to an external service.
* Build results - results of every build should be published to a specific commit in github. Any failures should present which specific stage failed and show the test results in case of unit tests or security tests.
*  Notifications - notification to the committer (or to a relevant mailing list) should be sent after each build, especially in case of failure.
*  Logging - logs should be collected across the k8s cluster and should be published to a central location for log analysis and storage. Log Management Tools such as ELK, Sematext Logs, Datadog, Logz.io etc. should be used.

## API

Base URL will be dynamic according to the k8s distribution and the service type.
Details will be printed after installing the helm chart.
Default base URL: http://localhost:8080

| API endpoint | Method | Request Body | Server Response | Response Code |
|--------------|--------|--------------|-----------------|---------------|
| /api/v1/calculate | POST | {"expression": "2 * 2"} | {"result":"4"} | 200 |
| /api/v1/calculate | POST | "expression": "2 * 2" | {"error":"Bad request","error_message":"invalid request body"} | 400 |
| /api/v1/calculate | GET | {"expression": "2 * 2"} | Method Not Allowed | 405 |
| /coffee | | | I'm a teapot | 418 |
| /api/v1/tea | | | 404 page not found | 404 |

### Response Codes

- 200 - Successful request
- 400 - Bad request
- 404 - Resource not found
- 405 - Method not allowed
- 422 - Invalid expression (e.g. English letter instead of a number)
- 500 - Internal server error

### Usage Examples

For sending POST requests, it's most convenient to use [Postman](https://www.postman.com/downloads/).

1. StatusOK 200

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "42 + 5 * 2"
}'

# {"result":52}
```

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "6-8"
}'

# {"result":-2}
```

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "123(3/2)"
}'

# {"result":184.5}
```

2. Bad Request 400

```bash
curl 'localhost/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "2 * 2
}'

# {"error":"Bad request","error_message":"invalid request body"}
```

3. Unprocessable Entity 422

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "cat + 100500"
}'

# {"error":"Expression is not valid","error_message":"invalid characters in expression"}
```

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "()"
}'

# {"error":"Expression is not valid","error_message":"the brackets are empty"}
```

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "1/(2 - 3 + 1)"
}'

# {"error":"Expression is not valid","error_message":"division by zero is not allowed"}
```

```bash
curl 'localhost:8080/api/v1/calculate' \
--header 'Content-Type: application/json' \
--data '{
  "expression": "1 000 000 + 6"
}'

# {"error":"Expression is not valid","error_message":"missing operand"}
```
