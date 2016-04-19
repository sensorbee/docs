## Section markers

When defining a (sub)section, use the follow markers for each level:

* # with overline, for parts
* * with overline, for chapters
* =, for sections
* -, for subsections
* ^, for subsubsections
* ", for paragraphs

## Versioning

The documentation on the `master` branch describes the current SensorBee code on the `master` branch.

The documentation on the `vA.B` branch describes the latest SensorBee `vA.B.*` version.

When the SensorBee version increases on the bugfix level, then a new tag (e.g., `vA.B.D`) is added to [sensorbee/sensorbee](https://github.com/sensorbee/sensorbee) and the documentation should normally stay untouched (unless the bug is mentioned there).

When the SensorBee version increases on the minor or major level, then a new tag (e.g., `vA.C.0`) is added to [sensorbee/sensorbee](https://github.com/sensorbee/sensorbee) and a new branch `vA.C` is created on [sensorbee/docs](https://github.com/sensorbee/docs).

Where necessary, fixes on the ``master`` branch of the documentation (e.g., spelling mistakes) are backported to the branches.

![versioning scheme](https://raw.githubusercontent.com/sensorbee/docs/master/docs-versioning.png)

