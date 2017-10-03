FROM openshift/origin-ansible

MAINTAINER Andrew Block “andrew.block@redhat.com”

USER root

# Update System and install clients
RUN yum install -y epel-release; \
  yum update -y; \
  yum install -y --setopt=tsflags=nodocs https://repos.fedorapeople.org/repos/openstack/openstack-ocata/rdo-release-ocata-3.noarch.rpm; \
  yum install -y python-pip python-ceilometerclient \
                 python-cinderclient python-glanceclient \
                 python-heatclient python-neutronclient \
                 python-novaclient python-saharaclient \
                 python-swiftclient python-troveclient \
                 python-openstackclient python-dns \
                 pyOpenSSL;\
  yum clean all; \
  pip install --upgrade pip jinja2; \
  pip install shade six

COPY images/installer-openstack/root /

USER ${USER_UID}

CMD [ "/usr/local/bin/run_casl" ]
