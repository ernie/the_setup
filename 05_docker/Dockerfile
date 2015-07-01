FROM <my-prefix>/ruby-base

COPY docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

ENV RACK_ENV production
ENV BUNDLE_WITHOUT "test:development"
# You might need to add some bogus environment variables to pass checks by Rails
# initializers during asset precompilation.
# ENV DEVISE_SECRET_TOKEN replace_me_at_runtime

WORKDIR /app
ADD Gemfile /app/Gemfile
ADD Gemfile.lock /app/Gemfile.lock
RUN bundle install

ADD . /app/

RUN bundle exec rake assets:precompile --trace && \
      chown -R app /app

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
