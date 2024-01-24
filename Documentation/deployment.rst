Deployment guidelines
=====================

Re-signing SIGSTRUCT
--------------------

If you need to sign the enclave with another ("production") key after the
application was scaffolded, you can extract SIGSTRUCT from the SCAG-built
container. SIGSTRUCT is located in :file:`/app/app.sig` inside the container.

Ops people would then:

1. Extract the SIGSTRUCT from the container:

   .. code-block:: sh

        container=$(docker create "$image")
        docker cp "$container":/app/app.sig ./app.sig
        docker rm "$container"

2. Sign the SIGSTRUCT again. This step depends on the exact tooling and is not
   described in this document.

3. Replace the file in the docker image, e.g. by building a container inheriting
   from original container, just ADDing newly signed SIGSTRUCT. Note that the
   path inside the container MUST be :file:`/app/app.sig` again.

   .. code-block:: Dockerfile

        ARG FROM
        FROM ${FROM}
        COPY app.sig /app

   .. code-block:: sh

        docker build --build-arg=FROM="$image" .

   This newly built container may now be deployed as usual.
