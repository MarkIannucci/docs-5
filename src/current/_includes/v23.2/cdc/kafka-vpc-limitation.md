[VPC Peering](https://www.cockroachlabs.com/docs/cockroachcloud/network-authorization#vpc-peering) and [AWS PrivateLink](https://www.cockroachlabs.com/docs/cockroachcloud/network-authorization#aws-privatelink) in CockroachDB {{ site.data.products.dedicated }} clusters do **not** support connecting to a [Kafka]({% link {{ page.version.version }}/changefeed-sinks.md %}#kafka) sink's internal IP addresses for [changefeeds]({% link {{ page.version.version }}/change-data-capture-overview.md %}). To connect to a Kafka sink from CockroachDB {{ site.data.products.dedicated }}, it is necessary to expose the Kafka cluster's external IP address and open ports with firewall rules to allow access from a CockroachDB {{ site.data.products.dedicated }} cluster.