# Gitaly

GitLab uses [Gitaly](https://gitlab.com/gitlab-org/gitaly) to abstract all Git calls. To work on local changes to `gitaly`, please refer to the [Beginner's guide to Gitaly contributions](https://gitlab.com/gitlab-org/gitaly/blob/master/doc/beginners_guide.md).

### Options

By default, GDK is set up to talk to praefect as a proxy to gitaly. To disable praefect, use the `enabled` field under `praefect` in `gdk.yml`:

```yml
praefect:
  enabled: false
```
