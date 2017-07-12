# how do we build our elixir apps?

---

![containers](https://cdn.meme.am/instances/500x/63003519/sixth-sense-i-see-containers-everywhere.jpg)

---

## containers and buildkite

---

### containers

There's a container called `builder-elixir-1.4` that has useful stuff in it…

```bash
docker run -it --entrypoint /bin/bash 153929397925.dkr.ecr.us-west-2.amazonaws.com/builder-elixir-1.4
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

e.g. https://buildkite.com/blake-education/student-events/builds/2860

---

#### build

1. pulls the `builder-elixir-1.4` container from AWS ECR and runs the rest of these steps in that
2. clones the git repo at the right SHA for the build
3. runs `prepare_code` which does all the interesting work
4. `docker build` the container for deployment and `docker push` it to AWS ECR

https://github.com/blake-education/dockerfiles/blob/develop/app-builders/builder-base/build

---

##### prepare_code

1. installs `hex` and `rebar`
2. `mix deps.get` and `mix compile` for the *staging* and *prod* environments

There is also some work done to use a cache for the deps. That is a directory that exists on the EC2 build agent instance that is 'mounted' in the Docker container doing the build. At the end of the build process, those files are copied into the container because that cache-mount won't be available when running the container for real.

https://github.com/blake-education/dockerfiles/blob/develop/app-builders/builder-elixir-base/scripts/prepare_code
