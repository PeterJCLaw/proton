# Proton (3.0.0-rc2 [SemVer](http://semver.org/))

Proton is a protocol for Student Robotics match scoring scripts.

## Execution

1. A proton compliant program MUST be either a script including a `#!` line
   or be a binary.

2. A proton compliant program MUST be capable of being exec'd on a supported
   platform. This typically means that it has the executable bit set.

3. A proton compliant program MUST run on Linux.

4. A proton compliant program MUST NOT have any side effects beyond printing
   to STDERR, STDOUT and having a return code. This precludes:
   modifying its input, writing to a database or any other kind of I/O.

5. A proton compliant program MUST always give the same output for a given
   input.

## Definitions

1. A TLA is defined as a string matching the regex `[a-zA-Z]{3}[a-zA-Z0-9]*`.
2. A ZONE is defined as an integer between 0 and 3 inclusive.

## Inputs

1. A proton compliant program MUST consume a single argument which is the
   path to a YAML file containing a computerised interpretation of a Student
   Robotics scoresheet.

    1.0 A proton compliant program MUST be able to read a file from both
        absolute and relative paths.

    1.1 A proton compliant program MUST accept YAML files with the proton
        format, these are of the form:

    ~~~
    match_number: integer
    arena_id: integer or string representing arena identity
    teams: dictionary with 2-4 key value pairs:
        TLA: dictionary with key value pairs:
            zone: a ZONE
            disqualified: an optional boolean, defaulting to false
            present: an optional boolean, defaulting to true

            optionally other key value pairs representing data about
            scoring specific to the year and game which are most suitably
            stored against a team.

    arena_zones: optional dictionary with key value pairs:
        ZONE: optional dictionary
            any key value pairs representing data about scoring specific
            to the year and game which are most suitably stored against
            an arena zone.
        other: optional dictionary
            any key value pairs representing data about scoring specific
            to the year and game which are most suitably stored against
            the arena but not a team's zone.

    other: optional key containing any other content desired to be recorded
           about the results of the match. A typical use-case might be for
           storing non-teams related data for further validation.
    ~~~

    1.2 A proton compliant program MUST exit with 1 if the input is malformed
        YAML or does not comply with rule 1.1

2. A proton compliant program MUST consume YAML from stdin if it is not
   given a filename.

3. A proton compliant program MAY refuse to process an input if it detects
   nested scoring values it is unable to process. If this occurs it MUST
   exit with 2.

## Outputs

1. A proton compliant program MUST print a YAML dictionary to STDOUT if it
   succeeds.


    1.1 The output must be of the form:

    ~~~yaml
    version: string representing version of the proton protocol implemented e.g: "1.0.0"
    match_number: an integer representing the match number
    arena_id: integer or string representing arena identity
    scores: dictionary with exactly as many keys there were teams in the input
        TLA: dictionary with exactly the key value pairs
            score: numeric value, representing team's score (game points).
            present: boolean, value from the input
            disqualified: boolean, value from the input
            zone: integer, the zone the team was in
    ~~~

2. A proton compliant program MUST exit with 0 if it succeeds.

3. A proton compliant program's output to STDOUT MUST be considered unusable if
   it fails.

4. A proton compliant program MAY print to STDERR under any circumstances.


## Examples

### Valid inputs

```yaml
match_number: 3
arena_id: A
teams:
    CLF:
     squares : [[1,2,1],[1,0,1],[0,0,0]]
    PSC:
     squares : [[0,0,0],[0,2,0],[0,p,0]]
    BGR:
     squares : [[0,0,0],[3,0,0],[0,0,0]]
    QEH1:
     squares : [[0,0,0],[6,0,0],[0,0,0]]
other: 42
```

```yaml
match_number: 4
arena_id: B
teams:
    CLF:
     present : false
     squares : [[1,2,1],[1,0,1],[0,0,0]]
    PSC:
     squares : [[0,0,0],[0,2,0],[0,p,0]]
    QEH:
     squares : [[0,0,0],[6,0,0],[0,0,0]]
```

### Invalid inputs

```yaml
match_number: 3
arena_id: A
teams:
    1:
     present : false
     squares : [[1,2,1],[1,0,1],[0,0,0]]
    PSC:
     squares : [[0,0,0],[0,2,0],[0,p,0]]
    BGR:
     squares : [[0,0,0],[3,0,0],[0,0,0]]
    QEH:
     squares : [[0,0,0],[6,0,0],[0,0,0]]
```

```yaml
match_number: 3
arena_id: B
teams:
    CLF:
     present : 1.0
     squares : [[1,2,1],[1,0,1],[0,0,0]]
    PSC:
     squares : [[0,0,0],[0,2,0],[0,p,0]]
    BGR:
     squares : [[0,0,0],[3,0,0],[0,0,0]]
    QEH:
     squares : [[0,0,0],[6,0,0],[0,0,0]]
```

### Valid responses

```
version: 1.0.0
match_number: 3
arena_id: A
scores:
    CLF:
        score: 41.0
        present: true
        disqualified: false
        zone: 0
    PSC:
        score: 12.0
        present: true
        disqualified: false
        zone: 1
    BGR:
        score: 7.0
        present: true
        disqualified: false
        zone: 2
    QEH:
        score: 18.0
        present: true
        disqualified: false
        zone: 3

# with exit code 0
```

```
# with exit code 1
```

```
# with exit code 2
```

### Invalid responses

No match number
```
version: 1.0.0
scores:
    CLF:
        score: 41.0
        present: true
        disqualified: false
    PSC:
        score: 12.0
        present: true
        disqualified: false
    QEH:
        score: 18.0
        present: true
        disqualified: false
    BGR:
        score: 7.0
        present: true
        disqualified: false
# with exit code 0
```

Many missing values
```
version: 1.0.0
scores:
# with exit code 0
```

Missing match number and arena ID
```
version: 1.0.0
scores:
    CLF:
        score: 41.0
        present: true
        disqualified: false
    PSC:
        score: 12.0
        present: true
        disqualified: false
    BGR:
        score: 7.0
        present: true
        disqualified: false
    QEH:
        score: 18.0
        present: true
        disqualified: false
# with exit code 1
```
