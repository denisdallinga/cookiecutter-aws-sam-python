
BASE := $(shell /bin/pwd)
CODE_COVERAGE = 72
PIPENV ?= pipenv

#################
#  Python vars	#
#################

EVENT ?= event_not_defined

#############
#  SAM vars	#
#############

NETWORK = ""

help:
	$(info ${HELP_MESSAGE})
	@exit 0

install:
	@${PIPENV} install

shell:
	@$(PIPENV) shell

deliver: install build package deploy

local-run: build invoke

build:
	${PIPENV} lock -r > ${BASE}/mcc_software_collector/requirements.txt
	sam build --debug

package: _check_lambda_bucket_definition
	sam package --s3-bucket $(LAMBDA_BUCKET) --output-template-file packaged.yaml

deploy:
	@sam deploy --template-file packaged.yaml --stack-name mcc-software-collector --capabilities CAPABILITY_IAM  --parameter-overrides `cat ${BASE}/properties/${STAGE}.properties`

invoke: _check_event_definition ##=> Run SAM Local function with a given event payload
	@sam local invoke --event ${EVENT} --env-vars ${BASE}/properties/local-dev.json

test: ##=> Run pytest
	@AWS_XRAY_CONTEXT_MISSING=LOG_ERROR $(PIPENV) run python -m pytest --cov . --cov-report term-missing --cov-fail-under $(CODE_COVERAGE) tests/ -v

#############
#  Helpers  #
#############

# EVENT="<name_and_path_of_event>" must be passed as ARG for target or else fail
ifndef EVENT
	$(error [!] EVENT env not defined...FAIL)
endif

ifeq ($(wildcard $(EVENT)),)
	$(error [!] '$(EVENT)' file doesn't exist)
endif

_check_lambda_bucket_definition:
	$(info [*] Checking whether bucket $(LAMBDA_BUCKET) is supplied...)

# LAMBDA_BUCKET="<bucket>" must be passed as ARG for target or else fail
ifndef LAMBDA_BUCKET
	$(error [!] LAMBDA_BUCKET env not defined...FAIL)
endif

_check_stage_definition:
	$(info [*] Checking whether bucket $(STAGE) is supplied...)

# STAGE="<stage>" must be passed as ARG for target or else fail
ifndef STAGE
	$(error [!] STAGE env not defined...FAIL)
endif

define HELP_MESSAGE
	Environment variables to be aware of or to hardcode depending on your use case:

	EVENT
		Default: not_defined
		Info: Environment variable to declare which event to supply for the invoke commands

	LAMBDA_BUCKET
		Default: not_defined
		Info: Environment variable to declare in which bucket to store where the packaged lambda

	Common usage:

	...::: Sets the environment variables for deployment :::...
	$ make set-env STAGE=stage

	...::: Installs all required packages as defined in the pipfile :::...
	$ make install

	...::: Spawn a virtual environment shell :::...
	$ make shell

	...::: Creates build environment for local sam invocation and preparing sam package and deploy :::...
	$ make build

	...::: Runs SAM package :::...
	$ make package LAMBDA_BUCKET=bucket-to-store-packaged-lambda-code-in

	...::: Runs SAM deploy :::...
	$ make deploy

	...::: Invokes lambda once with provided event :::...
	$ make invoke

	...::: Run Pytest under tests/ with pipenv :::...
	$ make test

	...::: Deliver application(runs install build package and deploy) :::...
	$ make deliver-local LAMBDA_BUCKET=bucket-to-store-packaged-lambda-code-in

	...::: Build and invoke locally (runs build and invoke) :::...
	$ make local-run EVENT=events/mcc_software_collector_event.json
endef
