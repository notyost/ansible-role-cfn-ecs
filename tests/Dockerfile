FROM mgage/docker-ansible
ARG ANSIBLE_OPTIONS
ARG TEST_LABEL
ARG TEST_LABEL_KEY
ARG TEST_TAG
ARG GIT_COMMIT=unknown
LABEL $TEST_LABEL_KEY=$TEST_LABEL
LABEL git-commit=$GIT_COMMIT
LABEL TEST_TAG=$TEST_TAG
RUN apt-get update && apt-get install -y git &&\
    pip install boto boto3
ADD tests /tmp/playbook
ADD . /tmp/playbook/roles/$TEST_LABEL
WORKDIR /tmp/playbook
RUN ansible-galaxy install -r requirements.yml -p ./roles/ &&\
    ansible-playbook $ANSIBLE_OPTIONS -i inventory test.yml
