---
description: Using Grafana Loki with GoQuorum
---

# Grafana Loki

[Grafana Loki] is an open-source log management platform that is available when using the
[Quorum Developer Quickstart](../../tutorials/quorum-dev-quickstart/index.md).

The [Promtail configuration] ingests logs at regular defined intervals and outputs them to [Loki] for storage.

The `pipeline configuration` in Promtail defines pipeline stages that can collate logs natively or using the JSON format.

!!! note

    If using the pipeline regex stage in `Promtail`, configuration must match your log format.

To view the GoQuorum Quickstart network logs in Loki:

1. [Start the Quorum Developer Quickstart with GoQuorum](../../tutorials/quorum-dev-quickstart/using-the-quickstart.md),
    selecting Loki monitoring.
1. Open the [`Grafana Loki address`](http://localhost:3000/d/Ak6eXLsPxFemKYKEXfcH/quorum-logs-loki?orgId=1&var-app=quorum&var-search=&from=now-15m&to=now) listed
    by the sample networks `list.sh` script.

    The logs display in Loki.

    ![Loki logs](../../images/dashboard_grafana_loki.png)

<!-- Links -->
[Promtail configuration]: https://github.com/ConsenSys/quorum-dev-quickstart/blob/master/files/common/promtail/promtail.yml
[Loki]: https://github.com/ConsenSys/quorum-dev-quickstart/blob/master/files/common/loki/loki.yml
[Grafana Loki]: https://grafana.com/oss/loki/
