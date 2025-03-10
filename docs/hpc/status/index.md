---
title: Wynton HPC Status
---

<!-- markdownlint-disable-file MD024 MD025 -->

# UCSF {{ site.cluster.name }} Status

## Queue Metrics

{% assign periods = "day,week,month,year" | split: ',' %}

<ul class="nav nav-pills">
{% for period in periods %}
  <li{% if period == periods[0] %} class="active"{% endif %}><a data-toggle="pill" href="#by-{{ period }}"><span style="font-weight: bold;">{{ period }}</span></a></li>
{% endfor %}
</ul>
<div class="tab-content" style="margin-top: 1ex;">
{% for period in periods %}
  <div id="by-{{ period }}" class="tab-pane fade in{% if period == periods[0] %} active{% endif %}">
    <img src="{{ site.assets.status_root_path }}/status/figures/queues-{{ period }}.png" alt="queues usage during the last {{ period }}"/><br>
    <img src="{{ site.assets.status_root_path }}/status/figures/gpuq-{{ period }}.png" alt="GPU queues usage during the last {{ period }}"/><br>
<!--    
    <img src="{{ site.assets.status_root_path }}/status/figures/grafana_storage_totals-{{ period }}.png" alt="BeeGFS I/O throughput during the last {{ period }}"/><br>
-->
  </div>
{% endfor %}
</div>


## Compute Nodes

<div id="hosttablediv">
<div id="hosttablemessage">All compute nodes are functional.</div>
</div>


## {{ site.cluster.name }} Grafana Dashboard

Detailed statistics on the file-system load and other cluster metrics can be found on the [{{ site.cluster.name }} Grafana Dashboard](https://mon.wynton.ucsf.edu/grafana).  To access this, make sure you are on the UCSF network.  Use your {{ site.cluster.name }} credential to log in.


## Current Incidents

{% include_relative incidents-current.md %}


## Upcoming Incidents

{% include_relative incidents-upcoming.md %}


## Past Incidents

{% assign years = "2023,2022,2021,2020,2019,2018" | split: ',' %}

<ul class="nav nav-pills">
{% for year in years %}
  <li{% if year == years[0] %} class="active"{% endif %}><a data-toggle="pill" href="#{{ year }}"><span style="font-weight: bold;">{{ year }}</span></a></li>
{% endfor %}
</ul>

<div class="tab-content" style="margin-top: 1ex;">
{% for year in years %}
<div id="{{ year }}" class="tab-pane {% if year == years[0] %}fadein active{% else %}fade{% endif %}">
<section markdown="1">
{% include_relative incidents-{{ year }}.md %}
</section>
</div>
{% endfor %}




<!-- DO NOT EDIT ANYTHING BELOW -->

<script src="https://d3js.org/d3.v3.min.js"><!-- ~150 kB --></script>
<script src="https://cdn.datatables.net/1.10.16/js/jquery.dataTables.min.js"><!-- ~80 kB --></script>
<script src="https://cdn.datatables.net/1.10.16/js/dataTables.bootstrap.min.js"><!-- 2 kB --></script>

<!-- markdownlint-disable-file MD052 -->
<script type="text/javascript" charset="utf-8">
d3.text("/hpc/assets/data/host_table.tsv", "text/csv", function(host_table) {
  // drop header comments
  host_table = host_table.replace(/^[#][^\r\n]*[\r\n]+/mg, '');
  host_table = d3.tsv.parse(host_table);

  d3.text("https://raw.githubusercontent.com/UCSF-HPC/wynton-slash2/master/status/qstat_nodes_in_state_au.tsv", "text/csv", function(host_status) {
    
    // drop header comments
    host_status = host_status.replace(/^[#][^\r\n]*[\r\n]+/mg, '');
    host_status = d3.tsv.parse(host_status);

    var tbody, tr, td, td_status;
    var value;
    var nodes = 0, gpu_nodes = 0;
    var cores = 0, gpu_cores = 0;
    var nodes_with_issues = 0, cores_with_issues = 0;
    var gpu_nodes_with_issues = 0, gpu_cores_with_issues = 0;
    var cores_node;
    var hostname;
    
    /* For each row */
    host_table.forEach(function(row) {
      hostname = row["Node"];
      
      nodes += 1;
      cores_node = parseInt(row["Physical Cores"]);
      cores += cores_node;

      if (hostname.includes("gpu")) {
        gpu_nodes += 1;
        gpu_cores += cores_node;
      }
      
      // No issues?
      if (host_status.filter(function(d) { return d.queuename == hostname }).length == 0) return;

      /* Ignore column on /tmp size, iff it exists */
      delete row["Local `/tmp`"];

      if (nodes_with_issues == 0) {
        var table = d3.select("#hosttablediv").append("details").append("table");
        table.id = "hosttable";
        tr = table.append("thead").append("tr");
        tr.append("th").text("Status");
        for (key in row) tr.append("th").text(key.replace(/\`/g, ""));
        tbody = table.append("tbody");
      }

      nodes_with_issues += 1;      
      cores_with_issues += cores_node;
      if (hostname.includes("gpu")) {
        gpu_nodes_with_issues += 1;      
        gpu_cores_with_issues += cores_node;
      }
  
      tr = tbody.append("tr");
      td_status = tr.append("td").text("⚠");  // "⚠" or "✖"
      for (key in row) td = tr.append("td").text(row[key]);
    });


    /* WORKAROUND: The host table is not updates; instead pull in the static information. /HB 2020-12-16 */
    nodes = {{ site.data.specs.nodes }};
    cores = {{ site.data.specs.physical_cores }};
    gpu_nodes = {{ site.data.specs.gpu_nodes }};
    
    var div = document.getElementById("hosttablemessage");
    if (nodes_with_issues > 0) {
      var text = document.createTextNode("Currently, " + nodes_with_issues + " (" + (100*nodes_with_issues/nodes).toFixed(1) + "%) " +  " of " + nodes + " compute nodes, corresponding to " + cores_with_issues + " (" + (100*cores_with_issues/cores).toFixed(1) + "%) " + " of " + cores + " CPU cores, are unavailable. Out of these, " + gpu_nodes_with_issues + " (" + (100*gpu_cores_with_issues/gpu_cores).toFixed(1) + "%) of " + gpu_nodes + " GPU compute nodes are unavailable.");
      div.appendChild(text);
      text = document.createTextNode(" A compute node is considered unavailable when its queuing state is \"unheard/unreachable\" or \"error\" (according to ");
      div.appendChild(text);
      var code = document.createElement("code");
      code.innerText = "qstat -f -qs uE";
      div.appendChild(code);
      text = document.createTextNode(" queried every five minutes), which means they will not take on any new jobs.");
      div.appendChild(text);
    } else {
      var text = document.createTextNode("All " + nodes + " nodes, with a total of " + cores + " cores, are functional.");
      div.appendChild(text);
    }
    
    $(document).ready(function() {
      $('#hosttable').DataTable({
        paging: false,
        searching: false,
        order: [[ 1, "asc" ]]
      });
    });
  });
});
</script>
