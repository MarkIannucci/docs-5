In this example, you'll set up a core changefeed for a single-node cluster that emits Avro records. CockroachDB's Avro binary encoding convention uses the [Confluent Schema Registry](https://docs.confluent.io/current/schema-registry/docs/serializer-formatter.html) to store Avro schemas.

1. Use the [`cockroach start-single-node`]({% link cockroachcloud/cockroach-start-single-node.md %}) command to start a single-node cluster:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ cockroach start-single-node \
    --insecure \
    --listen-addr=localhost \
    --background
    ~~~

1. Download and extract the [Confluent Open Source platform](https://www.confluent.io/download/).

1. Move into the extracted `confluent-<version>` directory and start Confluent:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ ./bin/confluent start
    ~~~

    Only `zookeeper`, `kafka`, and `schema-registry` are needed. To troubleshoot Confluent, see [their docs](https://docs.confluent.io/current/installation/installing_cp.html#zip-and-tar-archives).

1. As the `root` user, open the [built-in SQL client](https://www.cockroachlabs.com/docs/{{ site.current_cloud_version }}/cockroach-sql):

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ cockroach sql --url="postgresql://root@127.0.0.1:26257?sslmode=disable" --format=csv
    ~~~

    {% include {{ page.version.version }}/cdc/core-url.md %}

    {% include {{ page.version.version }}/cdc/core-csv.md %}

1. Enable the `kv.rangefeed.enabled` [cluster setting]({% link cockroachcloud/cluster-settings.md %}):

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    > SET CLUSTER SETTING kv.rangefeed.enabled = true;
    ~~~

1. Create table `bar`:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    > CREATE TABLE bar (a INT PRIMARY KEY);
    ~~~

1. Insert a row into the table:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    > INSERT INTO bar VALUES (0);
    ~~~

1. Start the core changefeed:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    > EXPERIMENTAL CHANGEFEED FOR bar WITH format = experimental_avro, confluent_schema_registry = 'http://localhost:8081';
    ~~~

    ~~~
    table,key,value
    bar,\000\000\000\000\001\002\000,\000\000\000\000\002\002\002\000
    ~~~

1. In a new terminal, add another row:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ cockroach sql --insecure -e "INSERT INTO bar VALUES (1)"
    ~~~

1. Back in the terminal where the core changefeed is streaming, the output will appear:

    ~~~
    bar,\000\000\000\000\001\002\002,\000\000\000\000\002\002\002\002
    ~~~

    Note that records may take a couple of seconds to display in the core changefeed.

1. To stop streaming the changefeed, enter **CTRL+C** into the terminal where the changefeed is running.

1. To stop `cockroach`, run:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ cockroach quit --insecure
    ~~~

1. To stop Confluent, move into the extracted `confluent-<version>` directory and stop Confluent:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ ./bin/confluent stop
    ~~~

    To terminate all Confluent processes, use:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ ./bin/confluent destroy
    ~~~
