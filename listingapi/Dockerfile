FROM python:3.8-slim

RUN useradd -ms  /bin/bash listingapi
COPY --chown=listingapi listingapi /home/listingapi/listingapi

USER listingapi
WORKDIR /home/listingapi/app

RUN pip3 install pipenv
COPY --chown=listingapi Pipfile Pipfile.lock ./

ENV PATH /home/listingapi/.local/bin:${PATH}
RUN pipenv install --deploy --system

ENV PYTHONPATH /home/listingapi/app
