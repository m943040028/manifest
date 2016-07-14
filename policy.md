# Policies for manifests

This document describes policies about we structure our manifests.

 * If members of the Fuchsia project do not have the authority to revert changes
   in a repository, that repository must be pinned to a specific git hash. This
   policy prevents changes in such a repository from breaking the Fuchsia build
   and having Fuchsia project members be unable to fix the build by reverting
   the offending change.

 * Repositories hosted on external servers, such has GitHub, should use a
   project with a `name` attribute of the following form:
   `external/<hostname>/path/to/repo`. This naming scheme ensures that multiple
   references to the same external repository have consistent manifest values.

 * Projects that contain non-standard licenses should use a `path` attribute
   that begins with `third_party/`.
