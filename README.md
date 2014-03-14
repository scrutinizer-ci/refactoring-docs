# Refactoring Documentation

This documentation contains a list of refactorings and examples with meta information for easy matching and
usage in Scrutinizer.

## Adding a Refactoring

To add a new refactoring, simply create a folder with the refactoring's name and place a meta.yml file in
that folder. See below for a template of that meta.yml:

```yml
# meta.yml
title: Some meaningful title
initial_situation: 'Short text describing the initial situation.'
target_situation: 'Short text of what this refactoring achieves.'

difficulty: easy    # Available difficulty: easy, medium, advanced

fixes: [long-method]   # Available smells: long-method, conditional-complexity, duplicated-code

intro: |
    A short overview of the refactoring, how it relates to other refactoring, 
    things/guidelines which should be considered.

    This text should be in rst format (restructured text).

pros:
    - 'Helps with A.'
    - 'Solves B.'

cons:
    - 'Could be bad if C.'
    - 'Sometimes leads to D.'

related: [extract-class]  # A list of related refactorings 
                          # (see folder names for the available ones)
```

## Adding an Example

To add an example, simply create a .rst (restructured text) document in the folder of the refactoring,
and add the example to the meta.yml in that folder:

```yml
# meta.yml
examples:
    'your-rst-file-name':    # Notice to leave off the .rst at the end here
        title: Some Meaningful Title
```

## License

This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/.

If you would like to use the work, it is greatly appreciated if you place a visible link on each 
sub-page which uses material of this documentation, and link back to scrutinizer-ci.com - thanks :)
