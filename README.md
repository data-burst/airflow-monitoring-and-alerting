# Airflow Monitoring and Alerting

<p align=center>
    <img src="./assets/images/diagrams.png" alt="Airflow Monitoring and Alerting"/>
</p>

<p align=center>
    <a href="https://github.com/data-burst/airflow-monitoring-and-alerting/graphs/contributors">
    <img src="https://img.shields.io/github/contributors-anon/data-burst/airflow-monitoring-and-alerting?color=yellow&style=flat-square" alt="contributors">
    </a>
    <a href="https://github.com/data-burst/airflow-monitoring-and-alerting/LICENSE"> 
    <img src="https://img.shields.io/badge/MIT-blue.svg?style=flat-square&label=license" alt="license">
</a>
</p>

## Table of Contents 🏗️

- [Airflow Monitoring and Alerting](#airflow-monitoring-and-alerting)
  - [Table of Contents 🏗️](#table-of-contents-️)
  - [Project Description 🌱](#project-description-)
  - [Project Usage 🧑‍💻](#project-usage-)
  - [Integration Steps for Airflow Monitoring Stack in Existing Setup :speech\_balloon:](#integration-steps-for-airflow-monitoring-stack-in-existing-setup-speech_balloon)
  - [Add statsd-exporter configuration to Helm values ☸️](#add-statsd-exporter-configuration-to-helm-values-️)
  - [Contributing 👥](#contributing-)
  - [License 📄](#license-)

## Project Description 🌱

This project offers a robust monitoring stack for Apache Airflow, encapsulated within Docker. It harnesses the capabilities of Prometheus, Grafana, StatsD, and Alertmanager to deliver real-time monitoring and visualization of your Airflow workflows.

The stack is designed for easy deployment and configuration. Grafana dashboards are automatically provisioned, enabling immediate visualization of key metrics from your Airflow instance. The project supports the addition of new dashboards by simply adding the corresponding JSON file to the `config_files/grafana/var/lib/grafana/dashboards` directory.

In addition to this, Alertmanager is incorporated into the stack to group alerts and send notifications to a Discord or Slack channel using a webhook token. This ensures you are always informed about the status of your workflows and can respond promptly to any issues.

## Project Usage 🧑‍💻

**Important Tip**:

Before you can use the project, based on [Airflow's documentation](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#setting-the-right-airflow-user), you need to ensure that Airflow has the correct permissions for the required directories. To do this, execute the following commands in the directory where your `docker-compose.yaml` file is located:

```bash
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

Deploying the project is straightforward:

1. **Clone the Repository**: Use git clone  to clone the repository onto your local machine.

    ```bash
    git clone https://github.com/data-burst/airflow-monitoring-and-alerting.git
    ```

2. Configure your Alertmanager by adding your Discord or Slack webhook token to the config_files/alertmanager/config.yaml file. Replace `<your_discord_webhook>` for Discord and `<your_slack_webhook>` for Slack. Additionally, update `<#your_slack_channel_name>` with your Slack channel name, for example, `#my_channel_name`."

3. **Start the Services**: Navigate to the project directory and use the following command to initiate all services in detached mode.

   ```bash
   docker compose up -d
   ```

   **P.S.**: Starting from Docker Compose V2, the docker compose command is integrated directly into the Docker CLI and Docker Engine1. This replaces the docker-compose feature from Docker Compose V11. Therefore, if you’re using Docker Compose V2 or later, you should be able to use the `docker compose` command, otherwise use `docker-compose`.

4. **Enjoy**: That’s it! Your Apache Airflow monitoring stack is now operational.

## Integration Steps for Airflow Monitoring Stack in Existing Setup :speech_balloon:

To integrate the monitoring stack for Apache Airflow into an existing setup with Airflow and Prometheus, follow these instructions:

1. **Install the required components**: Make sure you have Airflow and Prometheus and Grafana installed and configured in your environment.

2. **Mapping the configuration files**: Copy the contents of the `config_files/statsd/statsd.yaml` file from the monitoring project repository into your own Statsd configuration directory.

3. **Modify Airflow configuration**: Open your Airflow configuration file (airflow.cfg) and add the following lines:

   ```bash
   AIRFLOW__METRICS__STATSD_ON: "True"
   AIRFLOW__METRICS__STATSD_HOST: "statsd-exporter"
   AIRFLOW__METRICS__STATSD_PORT: "9125"
   AIRFLOW__METRICS__STATSD_PREFIX: "airflow"
   ```

    🔍️ Adjust the values based on your specific configuration.

4. **Configure Prometheus**: In your Prometheus configuration file (prometheus.yml), add the scrape configuration for the StatsD exporter:

    ```bash
    - job_name: 'statsd-exporter'
      scrape_interval: 15s
      static_configs:
        - targets:
          - 'statsd-exporter:9102'
    ```

    🔍️ Save the file after making the changes.

5. **Copy Grafana Dashboard JSON file**: From the `airflow-monitoring-and-alerting` repository, navigate to the `config_files/grafana/var/lib/grafana/dashboards` directory. Copy the JSON files of the desired dashboards you want to add to your Grafana instance.

*Note*: The specific directory path may vary depending on your Grafana setup.

By following these steps, you will integrate the monitoring stack from the `airflow-monitoring-and-alerting` repository into your existing Airflow and Prometheus setup. Make sure to adjust the configurations and paths according to your environment.

## Add statsd-exporter configuration to Helm values ☸️

If you are deploying the Airflow using [Airflow Helm chart](https://artifacthub.io/packages/helm/apache-airflow/airflow), you have to first enable stats statsd (`statsd.enabled: true`), and then add statsd-exporter configuration to your `extraMappings` list. Here is the configuration:

```yaml
statsd:
  enabled: true
  # other configurations
    extraMappings:
    - match: "(.+)\\.(.+)_start$"
      match_metric_type: counter
      name: "af_agg_job_start"
      match_type: regex
      labels:
        airflow_id: "$1"
        job_name: "$2"
    - match: "(.+)\\.(.+)_end$"
      match_metric_type: counter
      name: "af_agg_job_end"
      match_type: regex
      labels:
        airflow_id: "$1"
        job_name: "$2"
    - match: "(.+)\\.operator_failures_(.+)$"
      match_metric_type: counter
      name: "af_agg_operator_failures"
      match_type: regex
      labels:
        airflow_id: "$1"
        operator_name: "$2"
    - match: "(.+)\\.operator_successes_(.+)$"
      match_metric_type: counter
      name: "af_agg_operator_successes"
      match_type: regex
      labels:
        airflow_id: "$1"
        operator_name: "$2"
    - match: "*.ti_failures"
      match_metric_type: counter
      name: "af_agg_ti_failures"
      labels:
        airflow_id: "$1"
    - match: "*.ti_successes"
      match_metric_type: counter
      name: "af_agg_ti_successes"
      labels:
        airflow_id: "$1"
    - match: "*.zombies_killed"
      match_metric_type: counter
      name: "af_agg_zombies_killed"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler_heartbeat"
      match_metric_type: counter
      name: "af_agg_scheduler_heartbeat"
      labels:
        airflow_id: "$1"
    - match: "*.dag_processing.processes"
      match_metric_type: counter
      name: "af_agg_dag_processing_processes"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.tasks.killed_externally"
      match_metric_type: counter
      name: "af_agg_scheduler_tasks_killed_externally"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.tasks.running"
      match_metric_type: counter
      name: "af_agg_scheduler_tasks_running"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.tasks.starving"
      match_metric_type: counter
      name: "af_agg_scheduler_tasks_starving"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.orphaned_tasks.cleared"
      match_metric_type: counter
      name: "af_agg_scheduler_orphaned_tasks_cleared"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.orphaned_tasks.adopted"
      match_metric_type: counter
      name: "af_agg_scheduler_orphaned_tasks_adopted"
      labels:
        airflow_id: "$1"
    - match: "*.scheduler.critical_section_busy"
      match_metric_type: counter
      name: "af_agg_scheduler_critical_section_busy"
      labels:
        airflow_id: "$1"
    - match: "*.sla_email_notification_failure"
      match_metric_type: counter
      name: "af_agg_sla_email_notification_failure"
      labels:
        airflow_id: "$1"
    - match: "*.ti.start.*.*"
      match_metric_type: counter
      name: "af_agg_ti_start"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
        task_id: "$3"
    - match: "*.ti.finish.*.*.*"
      match_metric_type: counter
      name: "af_agg_ti_finish"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
        task_id: "$3"
        state: "$4"
    - match: "*.dag.callback_exceptions"
      match_metric_type: counter
      name: "af_agg_dag_callback_exceptions"
      labels:
        airflow_id: "$1"
    - match: "*.celery.task_timeout_error"
      match_metric_type: counter
      name: "af_agg_celery_task_timeout_error"
      labels:
        airflow_id: "$1"

    # === Gauges ===
    - match: "*.dagbag_size"
      match_metric_type: gauge
      name: "af_agg_dagbag_size"
      labels:
        airflow_id: "$1"
    - match: "*.dag_processing.import_errors"
      match_metric_type: gauge
      name: "af_agg_dag_processing_import_errors"
      labels:
        airflow_id: "$1"
    - match: "*.dag_processing.total_parse_time"
      match_metric_type: gauge
      name: "af_agg_dag_processing_total_parse_time"
      labels:
        airflow_id: "$1"
    - match: "*.dag_processing.last_runtime.*"
      match_metric_type: gauge
      name: "af_agg_dag_processing_last_runtime"
      labels:
        airflow_id: "$1"
        dag_file: "$2"
    - match: "*.dag_processing.last_run.seconds_ago.*"
      match_metric_type: gauge
      name: "af_agg_dag_processing_last_run_seconds"
      labels:
        airflow_id: "$1"
        dag_file: "$2"
    - match: "*.dag_processing.processor_timeouts"
      match_metric_type: gauge
      name: "af_agg_dag_processing_processor_timeouts"
      labels:
        airflow_id: "$1"
    - match: "*.executor.open_slots"
      match_metric_type: gauge
      name: "af_agg_executor_open_slots"
      labels:
        airflow_id: "$1"
    - match: "*.executor.queued_tasks"
      match_metric_type: gauge
      name: "af_agg_executor_queued_tasks"
      labels:
        airflow_id: "$1"
    - match: "*.executor.running_tasks"
      match_metric_type: gauge
      name: "af_agg_executor_running_tasks"
      labels:
        airflow_id: "$1"
    - match: "*.pool.open_slots.*"
      match_metric_type: gauge
      name: "af_agg_pool_open_slots"
      labels:
        airflow_id: "$1"
        pool_name: "$2"
    - match: "*.pool.queued_slots.*"
      match_metric_type: gauge
      name: "af_agg_pool_queued_slots"
      labels:
        airflow_id: "$1"
        pool_name: "$2"
    - match: "*.pool.running_slots.*"
      match_metric_type: gauge
      name: "af_agg_pool_running_slots"
      labels:
        airflow_id: "$1"
        pool_name: "$2"
    - match: "*.pool.starving_tasks.*"
      match_metric_type: gauge
      name: "af_agg_pool_starving_tasks"
      labels:
        airflow_id: "$1"
        pool_name: "$2"
    - match: "*.smart_sensor_operator.poked_tasks"
      match_metric_type: gauge
      name: "af_agg_smart_sensor_operator_poked_tasks"
      labels:
        airflow_id: "$1"
    - match: "*.smart_sensor_operator.poked_success"
      match_metric_type: gauge
      name: "af_agg_smart_sensor_operator_poked_success"
      labels:
        airflow_id: "$1"
    - match: "*.smart_sensor_operator.poked_exception"
      match_metric_type: gauge
      name: "af_agg_smart_sensor_operator_poked_exception"
      labels:
        airflow_id: "$1"
    - match: "*.smart_sensor_operator.exception_failures"
      match_metric_type: gauge
      name: "af_agg_smart_sensor_operator_exception_failures"
      labels:
        airflow_id: "$1"
    - match: "*.smart_sensor_operator.infra_failures"
      match_metric_type: gauge
      name: "af_agg_smart_sensor_operator_infra_failures"
      labels:
        airflow_id: "$1"

    # === Timers ===
    - match: "*.dagrun.dependency-check.*"
      match_metric_type: observer
      name: "af_agg_dagrun_dependency_check"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
    - match: "*.dag.*.*.duration"
      match_metric_type: observer
      name: "af_agg_dag_task_duration"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
        task_id: "$3"
    - match: "*.dag_processing.last_duration.*"
      match_metric_type: observer
      name: "af_agg_dag_processing_duration"
      labels:
        airflow_id: "$1"
        dag_file: "$2"
    - match: "*.dagrun.duration.success.*"
      match_metric_type: observer
      name: "af_agg_dagrun_duration_success"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
    - match: "*.dagrun.duration.failed.*"
      match_metric_type: observer
      name: "af_agg_dagrun_duration_failed"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
    - match: "*.dagrun.schedule_delay.*"
      match_metric_type: observer
      name: "af_agg_dagrun_schedule_delay"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
    - match: "*.scheduler.critical_section_duration"
      match_metric_type: observer
      name: "af_agg_scheduler_critical_section_duration"
      labels:
        airflow_id: "$1"
    - match: "*.dagrun.*.first_task_scheduling_delay"
      match_metric_type: observer
      name: "af_agg_dagrun_first_task_scheduling_delay"
      labels:
        airflow_id: "$1"
        dag_id: "$2"
```

## Contributing 👥

We welcome contributions to this repository! If you’re interested in contributing, please take a look at our [CONTIRIBUTION.md](./CONTRIBUTING.md) file for more information on how to get started. We look forward to collaborating with you!

## License 📄

This repository is licensed under the MIT License, which is a permissive open-source license that allows for reuse and modification of the code with few restrictions. You can find the full text of the license in [this](./LICENSE) file.
