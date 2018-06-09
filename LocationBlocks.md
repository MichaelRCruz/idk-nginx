# Location Blocks

The following four location blocks will provide a clear example of their usage and allude to the power of their utility.

### Prefix Match
Matches a URI starting with `/sup` followed by a short list of valid URIs.

```nginx
...

location /sup {
      return 200 'Greetings from "/sup"';
}

...
```

* `/sup`
* `/sup_my_dude/hi`
* `sup/hello`


### Exact Match
Will _only_ match the URI, `/sup` using the "equals" character.
```nginx
...

# Exact Match
location = /sup {
      return 200 'Greetings from "/sup" as an exact match';
}

...
```
### REGEX Match - case sensitive
Matches URIs based on regular expression using the tilde character.
```nginx
...

# REGEX Match - case sensitive
location ~ /sup[0-7] {
      return 200 'Greetings from "/sup" as a sensitive REGEX match';
}

...
```

The regular expression above will match any URI with `/sup` with an appended number, `/sup3`.

### REGEX Match - case insensitive
Matches URIs based on regular expression using an asterisk and a tilde character followed by a short list of valid URIs.
```nginx
...

# REGEX Match - case insensitive
location ~* /sup[0-7] {
      return 200 'Greetings from "/sup" as an insensitive REGEX match';
}

...
```
* `/Sup0`
* `/SUP3`
* `/sup7`

### Preferential Prefix Match
Note there is a default assignment to each location match modifier with a regular expression taking priority over a prefix match. You can supersede the regular expression match with the following example using the carrot tilde.

```nginx
...

# Preferential Prefix match
location ^~ /Sup2 {
      return 200 'Greetings from "/sup"';
}

...
```

### Order of Priorty

1. Exact Match `= uri`
2. Preferential Prefix Match `^~ uri`
3. REGEX Match `~* uri` – If a case sensitive and case insensitive match conflict, the first listed block takes priority.
4. Prefix Match `uri`
