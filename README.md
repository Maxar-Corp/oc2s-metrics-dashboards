# oc2s-metrics-dashboards

## Kibana

Kibana visualizations are generated from JSON files in the `kibana/visualizations`.

For specific template docs, see [this document].

### Quick Start: How to export and save your visualization

Follow these steps to export your new visualization or search from Kibana and add it to the play.

#### Steps: Exporting from Kibana
- In Kibana, go to **Management | Saved Objects | Dashboards/Searches/Visualizations**
- Find the visualization and check its box
- Click **Export** to download the file
- Rename the file
- Move the file into the appropriate subdirectory under _visualizations_

#### Steps: Adding a visualization to the play
- Open the _generate.yml_ file
- Add an item to the `kibana` list.

```yaml
# generate.yml
... other stuff ...

kibana:
  # Add your visualization here
  - visualization: Hello World
    file: visualizations/PATH_TO_HELLO_WORLD.json

```

#### Steps: Importing the visualizations into Kibana
- In Kibana, go to **Management | Saved Objects** and click **Import**
- Find the newly generated `all.json` file under the `output/` directory
- Select the `all.json` to import it (*You may be prompted to overwrite the old visualizations)

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
    index_pattern: omar-dev                      # Requires the index pattern
    validate: no                                 # Optionally validate (and pretty print) JSON
    
    kibana:
      - visualization: Test Visualization 1      # The title of your visualization
        file: visualizations/test1.json          # The path to the JSON file of your visualization

      - visualization: Test Visualization 2
        file: visualizations/test2.json
        description: This is a description!      # Add descriptions
        search: First Test Search                # Specify a search for the visualization by title
        
      - search: First Test Search                # Declare a "visualization", "search", or "dashboard"
        file: visualizations/search.json
    
      - search: Second Test Search
        file: visualizations/search.json
        columns:                                 # Optionally list columns of the search
          - foo
          - bar
          - foobar
          
      - dashboard: My Dashboard
        file: visualizations/my-dashboard.json
        panels:                                  # Specify dashboard panels by title
          - visualization: Test Visualization 1
          - visualization: Test Visualization 2
          - search: First Test Search
```

The example play contains visualizations, searches, and dashboards.

Another type of visualization is the _Template Visualization_. _Template Visualization_ may require other variables.

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
      "... other kibana stuff ...": ""
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
