web: bundle _2.4.6_ exec rails server -b 0.0.0.0 -p 3000
elastic: docker rm -f shiki-es >/dev/null 2>&1; exec docker run --rm --name shiki-es -p 9200:9200 -e discovery.type=single-node -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -v shiki-es-data:/usr/share/elasticsearch/data docker.elastic.co/elasticsearch/elasticsearch:6.8.23
webpack: bin/shakapacker-dev-server
neko: cd ../neko-achievements && SHIKIMORI_LOCAL=true mix run --no-halt
camo: CAMO_KEY=abc PORT=5566 CAMO_ENDPOINT_PATH=/ CAMO_SOCKET_TIMEOUT=90 CAMO_LENGTH_LIMIT=20971520 CAMO_LENGTH_LIMIT_REDIRECT=1 CAMO_403_REDIRECT=1 CAMO_LOGGING_ENABLED=debug CAMO_ALLOWED_HOSTS=safebooru.org,cdn.donmai.us,raikou1.donmai.us,raikou2.donmai.us,raikou3.donmai.us,raikou4.donmai.us,raikou5.donmai.us,danbooru.donmai.us,hijiribe.donmai.us,sonohara.donmai.us,yande.re,files.yande.re,assets.yande.re,konachan.com coffee ../camo-server/server.coffee
faye: cd ../faye-server && NODE_ENV=development FAYE_PORT=9292 FAYE_KEY=xxxxxxxxxxxxxxxxxxxx FAYE_ENDPOINT_PATH=/ node server.js
sidekiq: bundle exec sidekiq -C config/sidekiq.yml
# clockwork: bundle exec clockwork config/clock.rb
