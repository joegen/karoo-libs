.DEFAULT_GOAL := local-install

include config/utility.am

DOCKER = joegen/karoo-builder-rel7
PROJECTVER=@PACKAGE_VERSION@
DISTRO=x86_64
BASE_IMAGE=CentOS_7

PACKAGE_NAME = karoo-libs
REPO_FILE = oss_karoo_bridge_libs.repo

REPOHOST = ossapp.com
REPOUSER = ossappco
WWWROOT = /home5/ossappco/public_html
SSH_PASS = sshpass -p `cat .sshpass`

#REPOHOST = vmpool.gobitech.local
#REPOUSER = root
#SSH_PASS = 
#WWWROOT = /var/www/html

WWWREPOBASE = ${WWWROOT}/${PACKAGE_NAME}/${PROJECTVER}
WWWREPOPATH = ${WWWREPOBASE}/${BASE_IMAGE}/${DISTRO}
RPMPATH = ${HOME}/Devel/docker-build-root/${PACKAGE_NAME}/docker-build/RPMBUILD/RPMS/${DISTRO}

SSH_OPTIONS = -v -o UserKnownHostsFile=./.known_hosts -o StrictHostKeyChecking=no
SCP_RPM_PARAMS = ${RPMPATH}/* ${REPOUSER}@${REPOHOST}:${WWWREPOPATH}
SCP_REPOFILE_PARAMS = ${REPO_FILE} ${REPOUSER}@${REPOHOST}:${WWWREPOBASE}
CREATEREPO_PARAMS = ${REPOUSER}@${REPOHOST} createrepo ${WWWREPOPATH}
MKDIR_PARAMS = ${REPOUSER}@${REPOHOST} mkdir -p ${WWWREPOPATH}
RM_PARAMS = ${REPOUSER}@${REPOHOST} rm -rf ${WWWREPOPATH}/*

EXTERNALS_WWWREPOPATH = ${WWWREPOBASE}/externals/${BASE_IMAGE}/${DISTRO}/
EXTERNALS_RPMPATH = `cat .topsrc`/${BASE_IMAGE}/${DISTRO}
EXTERNALS_RM_PARAMS = ${REPOUSER}@${REPOHOST} rm -rf ${EXTERNALS_WWWREPOPATH}/*
EXTERNALS_SCP_RPM_PARAMS = ${EXTERNALS_RPMPATH}/* ${REPOUSER}@${REPOHOST}:${EXTERNALS_WWWREPOPATH}
	
SUBDIRS = .
	
# Require utility.am be included. Do not include it here because having all top-level makefile
# include nec. files helps readablity.  We have to avoid a network of includes that include other
# includes
utility_am_required ?= $(error Please include utility.am)
$(utility_am_required)

EXTRA_DIST = \
	config/revision-gen \
	config/doublequoteme \
	$(@PACKAGE@_EXTRA_DIST)

include libs/libs.am
include apps/apps.am
local-install: libs-install-local apps-install-local
install:  libs-install apps-install

dist-hook: @PACKAGE@-dist-revision @PACKAGE@-dist-specfile
	cp -rPp ${top_srcdir}/libs $(distdir)
	cp -rPp ${top_srcdir}/apps $(distdir)
	cd $(distdir)/libs && find . -name "*.spec" -print0 | xargs -0 rm -f
	cd $(distdir)/apps && find . -name "*.spec" -print0 | xargs -0 rm -f

%-dist-revision :
	echo $(PACKAGE_REVISION) > $(distdir)/.tarball-revision
%-dist-specfile :
	@$(call SearchAndReplace,$(srcdir)/@PACKAGE@.spec.in,$(distdir)/@PACKAGE@.spec, $(dist_EXTRA_REPLACE))

# CONFIG_DISTCLEANFILES are files to be added to DISTCLEANFILES in any
# makefile that includes config as an external subdirectory.
DISTCLEANFILES = \
	$(@PACKAGE@-DISTCLEANFILES) \
	config/config.sub \
	config/depcomp \
	config/install-sh \
	config/ltmain.sh \
	config/missing \
	Makefile.in \
	aclocal.m4 \
	autom4te.cache/output.0 \
	autom4te.cache/output.1 \
	autom4te.cache/requests \
	autom4te.cache/traces.0 \
	autom4te.cache/traces.1

rpm: dist
	rm -rf `pwd`/RPMBUILD; \
	mkdir -p `pwd`/RPMBUILD/{BUILD,SOURCES,RPMS,SRPMS,SPECS}; \
	rpmbuild -ta --define "%_topdir `pwd`/RPMBUILD" oss_karoo_bridge_libs*.gz

rpm-no-debug: dist
	rm -rf `pwd`/RPMBUILD; \
	mkdir -p `pwd`/RPMBUILD/{BUILD,SOURCES,RPMS,SRPMS,SPECS}; \
	rpmbuild -ta --define "%_topdir `pwd`/RPMBUILD" -D 'debug_package %{nil}' oss_karoo_bridge_libs*.gz	

top-src:
	export CURDIR=`pwd`; \
	cd ${top_srcdir}; echo `pwd` > ${CURDIR}/.topsrc; cd ${CURDIR};
	

docker-build-rpm: top-src
	rm -rf ${top_srcdir}/src/oss_core/build; \
	rm -rf ${top_srcdir}/src/kurento/build; \
	export PATH=$$PATH:/usr/local/bin; \
	if [ -f /usr/local/bin/docker-machine ]; then eval "$$(/usr/local/bin/docker-machine env default)"; fi; \
	docker kill oss-karoo-builder 2>/dev/null; \
	docker rm oss-karoo-builder 2>/dev/null; \
	docker pull ${DOCKER}; \
	docker run -t --network=host --rm --name oss-karoo-builder  -v `cat .topsrc`:/BUILDROOT -v ${HOME}:/BUILDHOME ${DOCKER} \
	/bin/sh -c "cd /BUILDROOT && ./docker-build-rpm";

docker-container:
	export PATH=$$PATH:/usr/local/bin; \
	if [ -f /usr/local/bin/docker-machine ]; then eval "$$(/usr/local/bin/docker-machine env default)"; fi; \
	docker build --no-cache -t ${DOCKER} .; \
	if [[ $$? -ne 0 ]]; then \
		exit 1; \
	fi; \
	docker push ${DOCKER};

deploy: top-src
	cp ${EXTERNALS_RPMPATH}/*.rpm ${RPMPATH}/
	createrepo ${RPMPATH}
	${SSH_PASS} ssh ${SSH_OPTIONS} ${MKDIR_PARAMS}; \
	if [[ $$? -ne 0 ]]; then \
		exit 1; \
	fi; \
	${SSH_PASS} ssh ${SSH_OPTIONS} ${RM_PARAMS}; \
	${SSH_PASS} scp ${SSH_OPTIONS} -r ${SCP_RPM_PARAMS}; \
	${SSH_PASS} scp ${SSH_OPTIONS} -r ${SCP_REPOFILE_PARAMS}; \
	if [[ $$? -ne 0 ]]; then \
		exit 1; \
	fi;

