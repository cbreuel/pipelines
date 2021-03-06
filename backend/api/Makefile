# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


# Building requires protobuf version 3.6.0 or above. Unzip this directory to a
# suitable location and modify the values of the following two variables before
# running make.
protocompiler = ${HOME}/protobuf-3.6.1/bin/protoc
protoinclude = ${HOME}/protobuf-3.6.1/include

TMPDIR := $(shell mktemp -d)

default: all

dependencies:
	@echo "Using temporary directory: $(TMPDIR)"
	GOBIN=$(TMPDIR) go install ../../vendor/github.com/go-swagger/go-swagger/cmd/swagger
	GOBIN=$(TMPDIR) go install ../../vendor/github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
	GOBIN=$(TMPDIR) go install ../../vendor/github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
	GOBIN=$(TMPDIR) go install ../../vendor/github.com/golang/protobuf/protoc-gen-go

all: dependencies

	# Delete currently generated code.
	rm -r -f go_http_client/*
	rm -r -f go_client/*

	# Compile the *.proto files into *.pb.go (grpc client).
	$(protocompiler) -I$(protoinclude) -I/usr/local/include -I. \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/ \
		--plugin=protoc-gen-go=${TMPDIR}/protoc-gen-go \
		--go_out=plugins=grpc:go_client \
		*.proto

	# Compile the *.proto files into *.pb.gw.go (grpc client).
	$(protocompiler) -I$(protoinclude) -I/usr/local/include -I. \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/ \
		--plugin=protoc-gen-grpc-gateway=${TMPDIR}/protoc-gen-grpc-gateway \
		--grpc-gateway_out=logtostderr=true:go_client \
		*.proto

  # Compile the *.proto files into *.swagger.json (swagger specification).
	$(protocompiler) -I$(protoinclude) -I/usr/local/include -I. \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
		-I../../vendor/github.com/grpc-ecosystem/grpc-gateway/ \
		--plugin=protoc-gen-swagger=${TMPDIR}/protoc-gen-swagger \
		--swagger_out=logtostderr=true:swagger \
		*.proto

	# Compile the *.swagger.json into go REST clients.
	# Instructions to install the 'swagger' command are located here:
	# https://github.com/go-swagger/go-swagger
	# swagger v0.17.0 was last used.

	$(TMPDIR)/swagger generate client \
		-f swagger/job.swagger.json \
		-A job \
		--principal models.Principal \
		-c job_client \
		-m job_model \
		-t go_http_client
	$(TMPDIR)/swagger generate client \
		-f swagger/run.swagger.json \
		-A run \
		--principal models.Principal \
		-c run_client \
		-m run_model \
		-t go_http_client
	$(TMPDIR)/swagger generate client \
		-f swagger/experiment.swagger.json \
		-A experiment \
		--principal models.Principal \
		-c experiment_client \
		-m experiment_model \
		-t go_http_client
	$(TMPDIR)/swagger generate client \
		-f swagger/pipeline.upload.swagger.json \
		-A pipeline_upload \
		--principal models.Principal \
		-c pipeline_upload_client \
		-m pipeline_upload_model \
		-t go_http_client
	$(TMPDIR)/swagger generate client \
		-f swagger/pipeline.swagger.json \
		-A pipeline \
		--principal models.Principal \
		-c pipeline_client \
		-m pipeline_model \
		-t go_http_client

	# Hack to fix an issue with go-swagger
	# See https://github.com/go-swagger/go-swagger/issues/1381 for details.
	sed -i -- 's/MaxConcurrency int64 `json:"max_concurrency,omitempty"`/MaxConcurrency int64 `json:"max_concurrency,omitempty,string"`/g' ./go_http_client/job_model/api_job.go
	sed -i -- 's/IntervalSecond int64 `json:"interval_second,omitempty"`/IntervalSecond int64 `json:"interval_second,omitempty,string"`/g' ./go_http_client/job_model/api_periodic_schedule.go
	sed -i -- 's/MaxConcurrency string `json:"max_concurrency,omitempty"`/MaxConcurrency int64 `json:"max_concurrency,omitempty,string"`/g' ./go_http_client/job_model/api_job.go
	sed -i -- 's/IntervalSecond string `json:"interval_second,omitempty"`/IntervalSecond int64 `json:"interval_second,omitempty,string"`/g' ./go_http_client/job_model/api_periodic_schedule.go

	# Executes the //go:generate directives in the generated code.
	go generate ./...

	# Add licenses to the generated files.
	# Instructions to install the 'autogen' command are located here:
	# https://github.com/mbrukman/autogen
	find ./ -name "*.go" -exec autogen.sh -i --no-tlc -c "Google LLC" -l apache {} \;

	@echo "Cleaning $(TMPDIR)"
	rm -r -f $(TMPDIR)