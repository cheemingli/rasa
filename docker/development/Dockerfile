FROM python:3.7-slim as base

# install libraries
RUN apt-get update -qq \
 && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git-core \
    graphviz-dev \
    libffi-dev \
    libffi6 \
    libpng-dev \
    libpq-dev \
    libssl-dev \
    openssh-client \
    openssl \
    pkg-config \
    wget \
 && apt-get autoremove -y

# install poetry
# keep this in sync with the version in pyproject.toml and Dockerfile
ENV POETRY_VERSION 1.0.3
RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
ENV PATH "/root/.poetry/bin:/opt/venv/bin:${PATH}"


FROM base as development-base

# copy poetry.lock pyproject.toml files to install dependencies 
# and to be able to use Docker layer cache
COPY poetry.lock pyproject.toml /code/

WORKDIR /code

# install dependencies
RUN python -m venv /opt/venv && \
  . /opt/venv/bin/activate && \
  pip install --no-cache-dir -U 'pip<20' && \
  poetry install --no-root --no-interaction


FROM development-base as tests

# install libraries
RUN apt-get install -y --no-install-recommends \
    graphviz \
    python-tk

ENV PIP_USE_PEP517 false

RUN . /opt/venv/bin/activate && \
    pip install spacy==2.1.9 --no-deps && \
    poetry install --extras spacy && \
    poetry run python -m spacy download en_core_web_md && \
    poetry run python -m spacy download de_core_news_sm && \
    poetry run python -m spacy link en_core_web_md en --force && \
    poetry run python -m spacy link de_core_news_sm de --force && \
    wget --progress=dot:giga -N -P data/ https://s3-eu-west-1.amazonaws.com/mitie/total_word_feature_extractor.dat

WORKDIR /code
COPY ./rasa ./rasa/
COPY ./README.md .

RUN . /opt/venv/bin/activate && \
    poetry install

# copy project to be able to run tests
COPY . .

ENTRYPOINT ["python", "-m", "pytest"]
CMD ["tests", "-n", "2", "--cov", "rasa"]


FROM development-base as development

# copy project
COPY . .
# remove data directory to be able to init
RUN rm -rf data

RUN python -m rasa init --no-prompt

ENTRYPOINT ["python", "-m", "rasa"]
CMD ["--help"]