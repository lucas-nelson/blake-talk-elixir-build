# how do we build our elixir apps?

---

![containers](https://cdn.meme.am/instances/500x/63003519/sixth-sense-i-see-containers-everywhere.jpg)

---

## containers and buildkite

---

### containers

There's a container called `builder-elixir-1.4` that has useful stuff in it…

```bash
docker \
  run \
  -it \
  --entrypoint /bin/bash \
  153929397925.dkr.ecr.us-west-2.amazonaws.com/builder-elixir-1.4
```

---

```bash
root@2ec3bff0a6fb:/# lsb_release -d
Description:	Ubuntu 14.04.4 LTS
root@40ab1a2c8c91:/# elixir -v
Erlang/OTP 19 [erts-8.2] [source-fbd2db2] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Elixir 1.4.1
```

---

### buildkite

The buildkite build consists of three phases:

1. build
2. test
3. release

<small>e.g. https://buildkite.com/blake-education/student-events/builds/2860</small>

---

#### build

Running on an AWS EC2 instance, the buildkite 'ci-builder-agent':

1. pull the `builder-elixir-1.4` container from AWS ECR and runs it to…
2. clone the git repo at the right SHA for the build
3. run `prepare_code` which does all the interesting work
4. `docker build` the container for deployment and `docker push` it to AWS ECR

<small>https://github.com/blake-education/dockerfiles/blob/develop/app-builders/builder-base/build</small>

---

##### prepare_code

1. install `hex` and `rebar`
2. `mix deps.get` and `mix compile`
3. for both the *staging* and *prod* environments

<small>https://github.com/blake-education/dockerfiles/blob/develop/app-builders/builder-elixir-base/scripts/prepare_code</small>

---

There is also work done to use a cache for the deps.

* a directory on the builder-agent is 'mounted' in the container
* after compilation, the cached files are copied into the container
* otherwise they won't be available when running the container for real

---

At this point the container that could be deployed to staging or production is complete. It's in ECR

---

#### test

Running on an AWS EC2 instance, the buildkite 'ci-tester-agent':

1. pull the newly build docker image from AWS ECR and runs it with a 'test' command to…
2. run the `scripts/docker/ci.sh`

<small>
https://github.com/blake-education/student_events/blob/develop/Dockerfile#L28
https://github.com/blake-education/student_events/blob/develop/script/docker/entrypoint.sh#L10-L12
https://github.com/blake-education/student_events/blob/develop/script/docker/ci.sh
</small>

---

##### ci.sh

1. compile for the `test` env
2. drop, create and migrate any databases it needs
3. `mix test`
4. `mix credo`

<small>https://github.com/blake-education/student_events/blob/develop/script/docker/ci.sh</small>

---

### release

Has been explained to me as "it just tags the container".

But the container was already tagged in the `prepare_code` stage: https://github.com/blake-education/dockerfiles/blob/develop/app-builders/builder-base/build#L96-L102

So yeah I'm not sure. This stage does not do much.

---

## what was all that?

1. the app is compiled both for staging and prod in a new container during the first stage of the build
2. the tests are run on the container, but no changes are uploaded to ECR
3. the unmodified container from step 1 is available for deploy
