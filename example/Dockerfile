# Start from base image (using Alpine Linux)
FROM ruby:2.6.5-alpine

LABEL maintainer="dlleal@uc.cl"

# Install necessary dependencies
RUN apk add --update --no-cache \
        openssl \
        tar \
        tzdata \
        postgresql-dev \
        build-base \
        nodejs \
    && wget https://yarnpkg.com/latest.tar.gz \
    && mkdir -p /opt/yarn \
    && tar -xf latest.tar.gz -C /opt/yarn --strip 1 \
    && rm -rf /var/lib/apt/lists/*

# Add Yarn binaries to the PATH
ENV PATH="$PATH:/opt/yarn/bin"

# Establish working directory
# (now every commands starts from /usr/src/app)
WORKDIR /usr/src/app

# Install node dependencies
RUN yarn install

# Copy Gemfile and Gemfile.lock to image
# (inside working directory /usr/src/app)
COPY Gemfile Gemfile.lock ./

# Install gems
RUN bundle install

# Copy script into /usr/bin
COPY entrypoint.sh /usr/bin/

# Make script executable
RUN chmod +x /usr/bin/entrypoint.sh

# Run script every time the container starts.
ENTRYPOINT ["entrypoint.sh"]

# Copy application files to image
# (the dot represents all files in the same folder
# relative to the Dockerfile inside your local machine)
COPY . /usr/src/app

# Precompile assets
RUN bundle exec rails assets:precompile

# Run this command when no command is specified
# (this will run on docker-compose up as default command)
CMD ["puma", "-C", "config/puma.rb"]
