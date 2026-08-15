# Copier JOLT

Copier JOLT is a JOLT-maintained fork of Copier for rendering and maintaining project templates.

> **Upstream credit:** This project began as a fork of [Copier](https://github.com/copier-org/copier).

**Repository:** <https://github.com/landfair-biz/copier-jolt>

Copier JOLT retains Copier's template rendering, answer management, Git-based template workflow, and update capabilities, while adding a more guided interactive prompting experience.

## What is different in Copier JOLT?

In addition to the standard Copier workflow, this fork adds the following features for template authors:

| Feature | What it adds |
| --- | --- |
| **Prompt navigation** | Let users review and revise previously entered interactive answers before generation continues. |
| **Inline prompt formatting** | Use bold text and prompt-toolkit colors inside `help` text and string defaults. |
| **Non-blocking warnings** | Give users contextual guidance without rejecting their answer. String warnings update while the user types. |
| **Question pre-tasks** | Run a trusted command before a question and use its output to build dynamic prompt choices. |
| **Dynamic question groups** | Repeat a set of related questions based on an earlier answer and save the results as a list of mappings. |

These additions are opt-in: templates without the new settings continue to use the familiar Copier behavior.

## Installation

Copier JOLT requires Python 3.10 or newer and Git 2.27 or newer.

Install the current version directly from this repository with your preferred Python tool:

```shell
pipx install git+https://github.com/landfair-biz/copier-jolt.git
```

```shell
uv tool install git+https://github.com/landfair-biz/copier-jolt.git
```

Or install it into an existing Python environment:

```shell
pip install git+https://github.com/landfair-biz/copier-jolt.git
```

The command-line program remains `copier`:

```shell
copier --help-all
```

## Quick start

A template is a directory containing a `copier.yml` (or `copier.yaml`) configuration file and the files to render.

```text
my-template/
├── copier.yml
├── {{ project_name }}/
│   └── README.md.jinja
└── {{ _copier_conf.answers_file }}.jinja
```

A minimal `copier.yml`:

```yaml
project_name:
  type: str
  help: What is your project name?

module_name:
  type: str
  help: What is your Python module name?
```

A templated file, `{{ project_name }}/README.md.jinja`:

```markdown
# {{ project_name }}

Python package: `{{ module_name }}`
```

Generate a project from a local template:

```shell
copier copy path/to/my-template path/to/new-project
```

Templates may also be Git URLs or shortcuts such as `gh:organization/template`. To use this fork itself as a template source, use:

```shell
copier copy https://github.com/landfair-biz/copier-jolt.git path/to/destination
```

You can also call Copier JOLT from Python:

```python
from copier import run_copy

run_copy("path/to/my-template", "path/to/new-project")
```

## JOLT prompt features

Add these settings to question dictionaries in `copier.yml`.

### Review and edit answers

Set `prompt_navigation: true` at the top level of your template configuration to offer an answer-review step after the interactive questionnaire.

```yaml
prompt_navigation: true

project_name:
  type: str
  help: Name of the new project

license:
  type: str
  choices:
    - MIT
    - Apache-2.0
    - Proprietary
```

After answering the prompts, Copier JOLT asks whether the user wants to review or edit answers. If they do, it shows the questions answered during that session. Choosing a question returns to that point in the questionnaire and clears the answers from that question onward so dependent questions can be answered again.

This is designed for interactive use. Non-interactive runs that provide `--defaults`, `--data`, or `--data-file` continue as usual.

### Format help and default text

Use `[bold]...[/bold]` to emphasize text and `[color=COLOR]...[/color]` to apply a prompt-toolkit color. Formatting is supported in a question's `help` and in string `default` values.

```yaml
service_name:
  type: str
  help: "Choose a [bold]unique[/bold] name. Use [color=ansiblue]lowercase letters and dashes[/color]."
  default: "[color=ansigreen]my-service[/color]"
```

The prompt displays the styled help text and a formatted default preview. The markup is presentation-only: accepting the default above saves `my-service`, never `[color=ansigreen]my-service[/color]`.

Colors use prompt-toolkit color names. Common choices include `ansiblue`, `ansigreen`, `ansiyellow`, `ansired`, `ansicyan`, and `ansimagenta`.

Formatting can be combined:

```yaml
project_slug:
  type: str
  help: "[bold][color=ansiyellow]Required:[/color][/bold] use a lowercase URL-safe slug."
  default: "[bold]sample-project[/bold]"
```

Keep markup well-formed by closing every `[bold]` and `[color=...]` tag. The feature only affects `help` and string defaults; it does not change the values rendered into generated files.

### Give non-blocking warnings

Use `warning` when an input is allowed but merits attention. Like a `validator`, it is a Jinja template that renders an empty string when no warning applies. Unlike a validator, it never blocks the user from proceeding.

```yaml
project_name:
  type: str
  help: Project name
  warning: >-
    {% if project_name | length < 5 %}
    Short names can be hard to discover later.
    {% endif %}
```

For string questions without `choices`, the warning appears in the prompt's bottom toolbar while the user types and disappears as soon as the condition is resolved. Choice prompts show the warning alongside each option that triggers it. In this example, the message clears once the name has five or more characters.

Warnings can guide users about conventions without forbidding exceptions:

```yaml
database_url:
  type: str
  help: Database connection URL
  warning: >-
    {% if database_url.startswith('postgres://') %}
    Prefer the postgresql:// scheme for new connections.
    {% endif %}
```

Use `validator` for requirements that must prevent submission, such as an invalid project slug; use `warning` for advice and review cues. For non-string prompts or string prompts with choices, a warning is displayed after the answer is submitted.

### Build choices from a command

A question can define `pre_tasks`: commands that run immediately before that question is displayed. Each task must have a `name` and a `command`. Its result becomes available to the question's templates at `_pre_tasks.<name>`.

The result exposes:

- `stdout` — complete standard output as text
- `stderr` — complete standard error as text
- `lines` — standard output split into newline-separated values

For example, list network interfaces and offer them as choices:

```yaml
network_interface:
  type: str
  help: Select the interface this service should bind to.
  pre_tasks:
    - name: interfaces
      command: [sh, -c, "ls /sys/class/net"]
  choices: |
    {% for interface in _pre_tasks.interfaces.lines %}
    - {{ interface }}
    {% endfor %}
```

Use a pre-task to derive choices from a project tool as well:

```yaml
package_manager:
  type: str
  pre_tasks:
    - name: available
      command: [sh, -c, "printf 'uv\npip\npoetry\n'"]
  choices: |
    {% for manager in _pre_tasks.available.lines %}
    - {{ manager }}
    {% endfor %}
```

Commands can be a shell-style string or an argument list. You can conditionally run a task with `when`:

```yaml
deployment_region:
  type: str
  pre_tasks:
    - name: regions
      command: [sh, -c, "./scripts/list-regions"]
      when: "{{ cloud_provider == 'aws' }}"
  choices: |
    {% for region in _pre_tasks.regions.lines %}
    - {{ region }}
    {% endfor %}
```

> **Security:** Pre-tasks execute commands with the same permissions as the person running Copier. Only use templates you trust. Pre-tasks use the same trusted-template approval flow as normal template tasks and are skipped when running with `--skip-tasks`.

### Ask a dynamic group of questions

Use a dynamic question group when an earlier response determines how many similarly structured records you need. Define `repeat` with a value or Jinja expression that resolves to a non-negative integer, then define the fields for each entry in `questions`.

```yaml
elasticsearch_node_count:
  type: int
  help: How many Elasticsearch nodes are in the cluster?

elasticsearch_nodes:
  repeat: "{{ elasticsearch_node_count }}"
  questions:
    hostname:
      type: str
      help: "Hostname for node {{ _repeat.number }} of {{ _repeat.count }}"
    mac_address:
      type: str
      help: "MAC address for node {{ _repeat.number }}"
    ip_address:
      type: str
      help: "IP address for node {{ _repeat.number }}"
```

For a count of `2`, the answers file receives one list under `elasticsearch_nodes`:

```yaml
elasticsearch_nodes:
  - hostname: es-1
    mac_address: "00:11:22:33:44:55"
    ip_address: 10.0.0.11
  - hostname: es-2
    mac_address: "00:11:22:33:44:66"
    ip_address: 10.0.0.12
```

Inside repeated prompts, `_repeat.index` is zero-based, `_repeat.number` is one-based, `_repeat.count` is the total number of records, and `_repeat.group` is the group name. The resulting list is available to files and later prompts as `elasticsearch_nodes`.

## Combining the features

The following example uses navigation, formatted guidance, a live warning, and dynamic choices together:

```yaml
prompt_navigation: true

cloud_provider:
  type: str
  help: Choose where this service will run.
  choices:
    - AWS
    - Local

region:
  type: str
  help: "Select a [bold]deployment region[/bold]."
  pre_tasks:
    - name: regions
      command: [sh, -c, "./scripts/list-regions"]
      when: "{{ cloud_provider == 'AWS' }}"
  choices: |
    {% if cloud_provider == 'AWS' %}
    {% for region_name in _pre_tasks.regions.lines %}
    - {{ region_name }}
    {% endfor %}
    {% else %}
    - local
    {% endif %}

service_slug:
  type: str
  help: "Use [color=ansicyan]lowercase letters, digits, and dashes[/color]."
  default: "[color=ansigreen]my-service[/color]"
  validator: >-
    {% if not (service_slug | regex_search('^[a-z][a-z0-9-]+$')) %}
    Start with a lowercase letter; use only lowercase letters, digits, and dashes.
    {% endif %}
  warning: >-
    {% if service_slug | length < 8 %}
    A longer service slug is usually easier to identify in logs.
    {% endif %}
```

Here, invalid slugs are rejected by `validator`, short-but-valid slugs show an advisory warning, the available regions depend on the provider, and the final review permits corrections before rendering.

## Core Copier capabilities

Copier JOLT continues to support the established Copier workflow:

- Render template variables in file contents, filenames, and directory names.
- Store answers in an answers file for repeatable generation and updates.
- Use local templates, Git repositories, Git URLs, and `gh:`/`gl:` shortcuts.
- Pass values non-interactively with `--data` or a data file.
- Recopy projects or update generated projects as their templates evolve.
- Use Jinja expressions, conditions, validation, choices, secrets, and tasks in template configuration.

For all command options, run:

```shell
copier --help-all
```

## Contributing and license

Issues, feature requests, and contributions belong in the [Copier JOLT repository](https://github.com/landfair-biz/copier-jolt).

Copier JOLT is distributed under the MIT License. See [LICENSE](LICENSE) for details.
