build:
	$(MAKE) check-repo-env
	$(MAKE) check-ncs-run-env
	$(MAKE) check-ncs-env
	$(MAKE) clean
	mkdir ${NCS_RUN_DIR}
	$(MAKE) netsim
	$(MAKE) setup
	$(MAKE) configure
	$(MAKE) compile
	$(MAKE) start

netsim:
	ncs-netsim --dir ${NCS_RUN_DIR}/netsim create-network ${NCS_DIR}/packages/neds/cisco-ios-cli-3.8 3 ex

setup:
	ncs-setup --dest ${NCS_RUN_DIR} --netsim-dir ${NCS_RUN_DIR}/netsim
	ncs-make-package --service-skeleton template learn-yang --dest ${NCS_RUN_DIR}/packages/learn-yang

configure:
	cp ${REPO_DIR}/learn-yang/Makefile ${NCS_RUN_DIR}/packages/learn-yang/src/
	cp ${REPO_DIR}/learn-yang/learn-yang.yang ${NCS_RUN_DIR}/packages/learn-yang/src/yang/
	cp ${REPO_DIR}/learn-yang/learn-yang-template.xml ${NCS_RUN_DIR}/packages/learn-yang/templates/

compile:
	$(MAKE) -C ${NCS_RUN_DIR}/packages/learn-yang/src clean all 

start:
	ncs-netsim -a start --dir ${NCS_RUN_DIR}/netsim
	cd ${NCS_RUN_DIR} && ncs --with-package-reload && cd -

clean:
	$(MAKE) stop
	-rm -rf ${NCS_RUN_DIR}

stop:
	$(MAKE) check-repo-env
	$(MAKE) check-ncs-run-env
	$(MAKE) check-ncs-env
	-ncs-netsim --dir ${NCS_RUN_DIR}/netsim -a stop
	-ncs --stop

check-repo-env:
ifndef REPO_DIR
	$(Error environment variable REPO_DIR is undefined. Source it. See example in README)
endif

check-ncs-run-env:
ifndef NCS_RUN_DIR
	$(Error environment variable NCS_RUN_DIR is undefined. Source it. See example in README)
endif

check-ncs-env:
ifndef NCS_DIR
	$(Error environment variable NCS_DIR is undefined. Source it. See example in README)
endif