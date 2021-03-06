# sphinx-doc - Docker image

This is a simple little Docker image intended to allow reasonably easy
generation of documentation using the Sphinx Documentation tool without having
to install the whole Latex bundle on your host machine.

This is certainly a fairly trivial way of using Docker, but there you go.

## Requirements

* Docker (obviously). Tested with 1.8.3 (build f4bf5c7) installed via Docker Machine on OSX (https://docs.docker.com/machine/)
* make (but only if you want to use the simple Makefile outlined below) - other build tools are available.

## Docker Hub

You should be able to pull the image from Docker hub like this:

```bash
$ docker pull umbrellium/sphinx-doc
```

## Usage

I've been using it like this.

1. Create a new empty folder in which your documentation will live.
2. Within that folder create a new Dockerfile with the following contents:

    ```dockerfile
    FROM umbrellium/sphinx-doc:latest
    MAINTAINER Sam Mulube <sam@umbrellium.co.uk>

    CMD ["/bin/bash"]

    WORKDIR /doc
    ```
3. Then I've been using it with the following minimal Makefile:

```makefile
# Makefile to generate documentation output
#
# Targets:
# 	quickstart
# 	html
# 	pdf
# 	clean

BUILDDIR = docs
VOLUMEDIR = /doc
VOLUME = $(shell pwd)/$(BUILDDIR):$(VOLUMEDIR)
IMAGE = umbrellium/sphinx-doc

.PHONY: html pdf deploy quickstart clean

builddir:
	mkdir -p $(BUILDDIR)

quickstart: builddir
	docker run --rm -i -t -v $(VOLUME) $(IMAGE) sphinx-quickstart

html: builddir
	docker run --rm -v $(VOLUME) $(IMAGE) make html

pdf: builddir
	docker run --rm -v $(VOLUME) $(IMAGE) make latexpdf

clean: builddir
	docker run --rm -v $(VOLUME) $(IMAGE) make clean
```

So here I'm just using Make to invoke specific commands inside the Docker
container to generate our required output.

You can then use the image like this:

```bash
$ make quickstart
```

This task first creates the shared volume in which the documentation source and
output live, then starts the container and invokes `sphinx-quickstart`. This
interactive command is designed to initialise a new Sphinx project, so asks you
a bunch of questions about the project, which you'll have to answer as you see
fit, and at the end it will have created a new sphinx project within a folder
called `docs` within your current folder.

```bash
$ make html
```

As you'd imagine this command runs a separate make task within the container to
generate HTML documentation from your RST source which can then be found in the
`docs/_build/html` folder. It will fail if you try and run this before you've actually
created the sphinx project using `make quickstart`.

```bash
$ make pdf
```

Similarly the above command runs the `latexpdf` task inside the container to
generate PDF documentation to `docs/_build/latex`. It will fail if you try and run
this before you've actually created the sphinx project using `make quickstart`.

```bash
$ make clean
```

Is the command to clean up generated artefacts. It will fail if you try and run this
before you've actually created the sphinx project using `make quickstart`.

## Exclude generated artefacts from source control

When working on actual documentation using this tool, you probably want to exclude
any generated content from your source control tool. When using Git you can do this
with the following `.gitignore`.

```gitignore
docs/_build
```

Which will ignore the generated output (provided you selected the defaults
during the `sphinx-quickstart` initialisation step. If you selected a
non default setting you'll have to alter your `.gitignore`.

Other source control systems have similar mechanisms for excluding content.
