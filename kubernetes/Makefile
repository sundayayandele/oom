# Copyright © 2017 Amdocs, Bell Canada
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

PARENT_CHART := onap
COMMON_CHARTS_DIR := common
# FIXME OOM-765
ROOT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
OUTPUT_DIR := $(ROOT_DIR)/dist
PACKAGE_DIR := $(OUTPUT_DIR)/packages
SECRET_DIR := $(OUTPUT_DIR)/secrets

ifneq ($(SKIP_LINT),TRUE)
	HELM_LINT_CMD := helm lint
else
	HELM_LINT_CMD := echo "Skipping linting of"
endif

SUBMODS := robot aai
EXCLUDES := config oneclick readiness test dist helm $(PARENT_CHART) dcae $(SUBMODS)
HELM_CHARTS := $(filter-out $(EXCLUDES), $(sort $(patsubst %/.,%,$(wildcard */.)))) $(PARENT_CHART)

.PHONY: $(EXCLUDES) $(HELM_CHARTS)

all: $(COMMON_CHARTS_DIR) $(SUBMODS) $(HELM_CHARTS) plugins

$(COMMON_CHARTS):
	@echo "\n[$@]"
	@make package-$@

$(HELM_CHARTS):
	@echo "\n[$@]"
	@make package-$@

$(SUBMODS):
	@echo "\n[$@]"
	@make submod-$@
	@make package-$@

submod-%:
	@make $*/requirements.yaml

%/requirements.yaml:
	$(error Submodule $* needs to be retrieved from gerrit.  See https://wiki.onap.org/display/DW/OOM+-+Development+workflow+after+code+transfer+to+tech+teams ); fi


make-%:
	@if [ -f $*/Makefile ]; then make -C $*; fi

dep-%: make-%
	@if [ -f $*/requirements.yaml ]; then helm dep up $*; fi

lint-%: dep-%
	@if [ -f $*/Chart.yaml ]; then $(HELM_LINT_CMD) $*; fi

package-%: lint-%
	@mkdir -p $(PACKAGE_DIR)
	@if [ -f $*/Chart.yaml ]; then helm package -d $(PACKAGE_DIR) $*; fi
	@helm repo index $(PACKAGE_DIR)

clean:
	@rm -f */requirements.lock
	@find . -type f -name '*.tgz' -delete
	@rm -rf $(PACKAGE_DIR)/*

# publish helm plugins via distrubtion directory
plugins:
	@cp -R helm $(PACKAGE_DIR)/

# start up a local helm repo to serve up helm chart packages
repo:
	@mkdir -p $(PACKAGE_DIR)
	@helm serve --repo-path $(PACKAGE_DIR) &
	@sleep 3
	@helm repo index $(PACKAGE_DIR)
	@helm repo add local http://127.0.0.1:8879

# stop local helm repo
repo-stop:
	@pkill helm
	@helm repo remove local
%:
	@:
