# **Follow the instructions step by step to deploy the stack to a swarm**

Here is the official [https://docs.docker.com/engine/swarm/stack-deploy/](https://docs.docker.com/engine/swarm/stack-deploy/) to practice.


## Set up a Docker registry

```sh
$ docker service create --name registry --publish published=5000,target=5000 registry:2
$ docker service ls
$ curl http://localhost:5000/v2/
```
## Create the example application

1. Create a new directory named `stackdemo` and navigate into it.

```sh
mkdir stackdemo
cd stackdemo
```

2. Create a new file named `app.py` and add the following code:

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```

3. Create a new file named `requirements.txt` and add the following content:

```plaintext
flask
redis
```

4. Create a new file named `Dockerfile` and add the following code:

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.4-alpine
COPY . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

Make sure to save all the files in the `stackdemo` directory.
- [X] compose.yml

```ts
services:
     web:
          image: 127.0.0.1:5000/stackdemo
          build: .
          ports:
                - "8000:8000"
     redis:
          image: redis:alpine
```
- terminal 

![Create](./images/test-app-creation.png)

##  Test the app with Compose

```sh
docker compose up -d

```
![Run](./images/test%20app%20with%20compose.png)

```sh
docker compose ps
curl http://localhost:8000
```
![Run](./images/run%20app.png)

```sh
docker compose down --volumes
```

## Push the generated image to the registry

```ts
docker compose push
```
![Push](./images/push%20image.png)
## Deploy the stack to the swarm

```sh
docker stack deploy --compose-file compose.yml stackdemo
docker stack services stackdemo
curl http://localhost:8000
```
![Deploy](./images/deploy%20swarm.png)

```sh
docker stack rm stackdemo
docker service rm registry
docker swarm leave --force
```
Happy Coding!!!

