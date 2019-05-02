ARG REGISTRY=presidio.azurecr.io

FROM python:3.7.1-alpine3.8 as presidio-python-deps

ARG re2_version="2018-12-01"
ARG NAME=presidio-analyzer

COPY ./${NAME}/Pipfile /usr/bin/${NAME}/Pipfile
COPY ./${NAME}/Pipfile.lock /usr/bin/${NAME}/Pipfile.lock

WORKDIR /usr/bin/${NAME}

RUN apk --update add --no-cache g++ && \
    apk --update add --no-cache --virtual build_deps make tar wget clang && \
    wget -O re2.tar.gz https://github.com/google/re2/archive/${re2_version}.tar.gz && \
    mkdir re2 && tar --extract --file "re2.tar.gz" --directory "re2" --strip-components 1 && \
    cd re2 && make install && cd .. && rm -rf re2 && rm re2.tar.gz && \
apk del build_deps

#RUN curl --silent https://bootstrap.pypa.io/get-pip.py | python3.7
RUN pip3 install pipenv
RUN pipenv install --sequential

FROM presidio-python-deps

ARG NAME=presidio-analyzer
WORKDIR /usr/bin/${NAME}
ADD ./${NAME} /usr/bin/${NAME}

RUN pipenv install --dev --sequential && \
    pipenv run pylint analyzer && \
    pipenv run flake8 analyzer --exclude "*pb2*.py" && \
	pipenv run pytest --log-cli-level=0

#----------------------------

FROM presidio-python-deps

ARG NAME=presidio-analyzer
ADD ./${NAME}/analyzer /usr/bin/${NAME}/analyzer
WORKDIR /usr/bin/${NAME}/analyzer

CMD pipenv run python __main__.py serve --env-grpc-port