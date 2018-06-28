# oc2s-metrics-dashboards

## Kibana

Kibana visualizations are generated from JSON files in the `kibana/visualizations`.

For specific template docs, see [this document].

### Quick Start: How to export and save your visualization

Follow these steps to export your new visualization or search from Kibana and add it to the play.

#### Steps: Exporting from Kibana
0. In Kibana, go to **Management | Saved Objects | Dashboards/Searches/Visualizations**
0. Find the visualization and check its box
0. Click **Export** to download the file
0. Rename the file
0. Move the file into the appropriate subdirectory under _visualizations_

#### Steps: Adding to the play
0. Open the _generate.yml_ file
0. Add an item to the `visualizations` list.

```yaml
# generate.yml
... other stuff ...

visualizations:
  # Add your visualization here
  - title: Hello World
    file: visualizations/PATH_TO_HELLO_WORLD.json

```

### Generating Visualizations

In order to generate the visualizations you must open the `kibana` directory and run the `generate.yml` Ansible playbook:

```console
$ ansible-playbook generate.yml
```

This will generate the visualizations listed in the play into the `output/` directory. For ease of importing into Kibana, a combined file including all of the generated visualizations will also be created in the same directory.

The play should look something like this:

```yaml
# generate.yml
---
- name: Generate Kibana visualizations
  hosts: localhost
  roles:
    - visualize

  vars:
    index_pattern: omar-dev # Required.
    validate: no # Optional. Defaults to 'yes'.
    
    visualizations:
      - title: test1
        file: "visualizations/test.json"

      - title: test2
        template: "visualizations/ingest/ingest-mission-id-count.json.j2"

      - title: "WFS Response Status Bar"
        template: "visualizations/generic/service/response-status-bar.json.j2"
        service: WFS
        http_status_key: httpStatus
```

The example play contains three visualizations

The first and second visualizations are simple _File Visualizations_ noted by the `file: "PATH"`. All visualizations require that a `title` and either a `file` or `template` variables be specified.

Some _Template Visualizations_ require other variables. The third visualization, _WFS Response Status Bar_, uses a template that additionally requires `service` and `http_status_key`.

Complex visualizations or dashboards can be specified depending on the template. Here is an example of one might specify such a template:

```json
... other variables ...

visualizations:
  # Some simple visualizations
  - title: WMS Popular Images
    file: "visualization/<...>/popular-images.json"
    
  - title: WMS Response Time
    template: "visualizations/<...>/response-time.json.j2"
    service: WMS
    response_time_key: responseTime
    
  # A dynamic dashboard template   
  - title: WMS Example Dashboard
    template: "visualizations/<...>/two-panel-dashboard.json.j2"
    panels:
      panel1: WMS Popular Images
      panel2: WMS Response Time

```

### Visualization Types

In the terms of this playbook, "visualization" is used to mean any kind of Kibana saved object (kibana visualization, dashboard, and search).

There are two types of visualizations: _File Visualizations_ and _Template Visualizations_.

#### File Visualizations

_File Visualizations_ are plain old JSON files exported from Kibana.

_Files Visualizations_ can container either a top level JSON object:

```json
{
  "_id": "cdc03c10-7593-11e8-bae2-613bc26f7fc6",
  "_type": "visualization",
  
  "... other kibana stuff ...": ""
}
```

Or a JSON list of visualizations:

```json
[
    {
      "_id": "cdc03c10-7593-11e8-bae2-613bc26f7fc6",
      ... other kibana stuff ...
    },
    {
      "_id": "cdc03c10-7593-11e8-bae2-613bc26f7fc6",
      "... other kibana stuff ...": ""
    }
]
```

The `_id` and `title` are overwritten to match the standard `id` pattern and use the user specified `title` as set in the play. 

**NOTE 1:**
_Currently you cannot use Jinja2 templates inside a File Visualization and they must contain only valid JSON. This restriction may be removed in the future._

**NOTE 2:**
_Currently dashboards are linked by ID and must be turned into templates with configurable links to visualizations. This restriction may be removed in the future when auto-id linking is added._

#### Template Visualizations

_Template Visualizations_ allow you to add variables to visualizations. Variables can be used to make visualizations more flexible and reusable than simple _File Visualizations_. Some generic visualizations with variables can be found in the visualization directory.

To make a _Template Visualization_ you must add a header and markers to your template file:

```jinja2
{% extends "kibana.j2" %}
{% block kibana %}
{
    "_id": "{{ getId(index_pattern, item.title) }}",
    "_type": "visualization",
    ... other kibana stuff ...
}
{% endblock kibana %}
```

- `{% extends "kibana.j2" %}` : This provides access to preset variables such as `id` and functions such as `getId(index, title)`.
- `{% block kibana %}` : This is required to mark the beginning of the visualization. The contents withing this block should be JSON and may contain Jinja2 templating.
- `{% endblock kibana %}` : This is required to mark the end of the visualization.
