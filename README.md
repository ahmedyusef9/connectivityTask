## Connectivity Task

instruction for run the task:
1. run :
`go run .`, the user can see the log in terminal.
   
2. run by _docker-compose_ : `docker-compose up --build` for detach the docker add `-d` flag.

this container run http server on port `5000`, the user able to send GET & POST request:


##### examples:

- POST :
  - `curl --location --request POST 'http://localhost:5000/address' \
    --header 'Content-Type: application/json' \
    --data-raw '[
    {
    "IP": "192.168.2.1",
    "Port": 8080
    },
    {
    "IP": "192.168.2.2",
    "Port": 80
    },
    {
    "IP": "192.168.2.3",
    "Port": 9000
    },
    {
    "IP": "127.0.0.1",
    "Port": 8080
    }]'`
  - response: POST REQUEST
- GET :
    - ``curl --location --request GET 'http://localhost:5000/address'``
    - response: `{"successful":[],"failed":[]}`