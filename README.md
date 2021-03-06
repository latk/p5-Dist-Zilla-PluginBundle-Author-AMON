# NAME

Dist::Zilla::PluginBundle::Author::AMON - dzil config choices by AMON

# SYNOPSIS

    # in dist.ini
    [@Author::AMON]
    github = example-user/example-repo
    autogenerate_file = Build.PL
    ; github_issues = 1
    ; exclude_author_deps = inc::Example
    ; gather_exclude_file = TODO.txt
    ; gather_exclude_match = ^scripts/
    ; ppport = src/ppport.h

# DESCRIPTION

This module contains a starting point for AMON's dist.ini.
There are a couple of more or less sensible defaults
(like pointing metadata at Github, or only gathering files tracked by Git)
and a number of valuable checks so that I mess up fewer CPAN releases
(like checking that the Changes file was updated.)
There are also a few convenience features
(like not gathering files that will be generated by the build process.)

Decisions not currently made by this module:

Which build system to use (e.g. Module::Build vs. MakeMaker).

How the version number is managed
(e.g. taken from the module with \[VersionFromMainModule\]).

How prereqs are found. Simply using \[AutoPrereqs\] is likely to be correct.

All this may change in a future release without prior notice,
and will not be seen as a backwards-incompatible change.

# CONFIGURATION

## github

The name of a GitHub repository in the format `<user>/<repo>`.
This is used to set the repo and user fields in
[\[GithubMeta\]](https://metacpan.org/pod/Dist::Zilla::Plugin::GithubMeta).

Required string.

## github\_issues

Whether Github Issues should be used as the bugtracker.
If not, bugtracker info needs to be added manually.

Optional boolean. Defaults to true.

## exclude\_author\_deps

Exclude authordeps prereqs.
This module will add your "dzil authordeps" via
[\[Prereqs::AuthorDeps\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Prereqs::AuthorDeps).
List any modules that should not be required,
e.g. because they are bundled as "inc/" modules.

Optional string. Can be specified multiple times. Defaults to empty list.

## gather\_exclude\_file

Filenames that should not be gathered.
Use [gather\_exclude\_match](#gather_exclude_match)
to exclude all files matching a pattern, e.g. a whole directory.
Use [autogenerate\_file](#autogenerate_file)
to exclude files that will be copied from the built dist.

Optional string. Can be specified multiple times. Defaults to empty list.

## gather\_exclude\_match

A regex to match filenames that should not be gathered.
Use [gather\_exclude\_file](#gather_exclude_file)
to exclude single files.
Use [autogenerate\_file](#autogenerate_file)
to exclude files that will be copied from the built dist.

Optional string. Can be specified multiple times. Defaults to empty list.

## autogenerate\_file

Files that should be copied from the built dist.
These won't be gathered.
After the build phase, they will be copied into your source tree.
Use [gather\_exclude\_file](#gather_exclude_file)
or [gather\_exclude\_match](#gather_exclude_match)
to exclude files, without copying them from the built dist.

The following files will be copied or autogenerated
without having to add them to this configuration option:

- `LICENSE` as generated by the \[License\] plugin.
- `cpanfile` as generated by the \[CPANFile\] plugin.
- `README.md` as generated by the \[ReadmeAnyFromPod\] plugin.
- `ppport.h` if specified (see the [ppport](#ppport) option).

Optional string. Can be specified multiple times. Defaults to empty list.

## ppport

Add a ppport.h file under the specified path.
The value of this option is the path/filename
where the ppport.h should be placed.
E.g.:

    ...
    ppport = src/ppport.h

Optional string or undef. Defaults to undef.

# PLUGINS

The plugins are configured in thematic groups.
Each group is a method that can be overridden in subclasses of this bundle.

- [configure\_meta](#configure_meta)
- [configure\_prereqs](#configure_prereqs)
- [configure\_gather](#configure_gather)
- [configure\_extra\_files](#configure_extra_files)
- [configure\_extra\_tests](#configure_extra_tests)
- [configure\_post\_build](#configure_post_build)
- [configure\_pre\_release](#configure_pre_release)
- (configure\_release)

    A minimum release workflow is hardcoded:

        [TestRelease]
        [ConfirmRelease]
        [UploadToCPAN]

- [configure\_post\_release](#configure_post_release)

## configure\_meta

Add metadata.

    [GithubMeta]
    user = {{github.user}}
    repo = {{github.repo}}
    issues = {{github_issues}}

Plugins:
[GithubMeta](https://metacpan.org/pod/Dist::Zilla::Plugin::GithubMeta).

Configuration options:
[github](#github),
[github\_issues](#github_issues).

## configure\_prereqs

Specify prerequisites.

    [Prereqs::AuthorDeps]
    ; exclude = {{exclude_author_deps}}

Plugins:
[Prereqs::AuthorDeps](https://metacpan.org/pod/Dist::Zilla::Plugin::Prereqs::AuthorDeps).

Configuration options:
[exclude\_author\_deps](#exclude_author_deps).

## configure\_gather

Gather files to be included in the dist.

    [Git::GatherDir]
    ; exclude_match = {{gather_exclude_match}}
    ; exclude_filename = {{gather_exclude_file}}
    ; exclude_filename = {{autogenerate_file}}
    ; exclude_filename = {{ppport}}
    [PruneCruft]

Plugins:
[Git::GatherDir](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::GatherDir),
[PruneCruft](https://metacpan.org/pod/Dist::Zilla::Plugin::PruneCruft).

Configuration options:
[gather\_exclude\_file](#gather_exclude_file),
[gather\_exclude\_match](#gather_exclude_match),
[autogenerate\_file](#autogenerate_file),
[ppport](#ppport).

## configure\_extra\_files

Autogenerate certain files in the dist.

    ; [NextRelease]
    [CPANFile]
    [MetaYAML]
    [MetaJSON]
    [MetaProvides::Package]
    [Manifest]
    [License]
    [Readme]
    ; [PPPort]
    ; filename = {{ppport}}

Plugins:
[\[CPANFile\]](https://metacpan.org/pod/Dist::Zilla::Plugin::CPANFile),
[\[MetaYAML\]](https://metacpan.org/pod/Dist::Zilla::Plugin::MetaYAML),
[\[MetaJSON\]](https://metacpan.org/pod/Dist::Zilla::Plugin::MetaJSON),
[\[MetaProvides::Package\]](https://metacpan.org/pod/Dist::Zilla::Plugin::MetaProvides::Package),
[\[Manifest\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Manifest),
[\[License\]](https://metacpan.org/pod/Dist::Zilla::Plugin::License),
[\[Readme\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Readme),
[\[PPPort\]](https://metacpan.org/pod/Dist::Zilla::Plugin::PPPort).

Configuration options:
[ppport](#ppport).

## configure\_extra\_tests

Add extra tests, particularly author tests focussed on release quality.

    [Test::Perl::Critic]
    [PodSyntaxTests]
    [PodCoverageTests]
    [Test::Kwalitee::Extra]
    [RunExtraTests]

Plugins:
[\[Test::Perl::Critic\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Test::Perl::Critic),
[\[PodSyntaxTests\]](https://metacpan.org/pod/Dist::Zilla::Plugin::PodSyntaxTests),
[\[PodCoverageTests\]](https://metacpan.org/pod/Dist::Zilla::Plugin::PodCoverageTests),
[\[Test::Kwalitee::Extra\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Test::Kwalitee::Extra),
[\[RunExtraTests\]](https://metacpan.org/pod/Dist::Zilla::Plugin::RunExtraTests).

## configure\_post\_build

Post-build actions, such as copying generated files into the source tree.

    [CopyFilesFromBuild
    copy = {{autogenerate_file}}
    ; copy = {{ppport}}
    [ReadmeAnyFromPod]
    type = markdown
    filename = README.md
    location = root
    phase = build

Plugins:
[\[CopyFilesFromBuild\]](https://metacpan.org/pod/Dist::Zilla::Plugin::CopyFilesFromBuild),
[\[ReadmeAnyFromPod\]](https://metacpan.org/pod/Dist::Zilla::Plugin::ReadmeAnyFromPod]).

Configuration options:
[autogenerate\_file](#autogenerate_file),
[ppport](#ppport).

## configure\_pre\_release

Run pre-release checks.

    [Git::CheckFor::CorrectBranch]
    [Git::Check]
    [CheckChangesHasContent]
    [CheckVersionIncrement]

Plugins:
[\[Git::CheckFor::CorrectBranch\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::CheckFor::CorrectBranch),
[\[Git::Check\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::Check),
[\[CheckChangesHasContent\]](https://metacpan.org/pod/Dist::Zilla::Plugin::CheckChangesHasContent),
[\[CheckVersionIncrement\]](https://metacpan.org/pod/Dist::Zilla::Plugin::CheckVersionIncrement).

## configure\_post\_release

Perform post-release bookkeeping.

    [NextRelease]
    time_zone = UTC
    [Git::Commit]
    commit_msg  = release-%v
    [Git::Tag]
    tag_format  = release-%v
    tag_message = release-%v
    [Git::Push]

Plugins:
[\[NextRelease\]](https://metacpan.org/pod/Dist::Zilla::Plugin::NextRelease),
[\[Git::Commit\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::Commit),
[\[Git::Tag\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::Tag),
[\[Git::Push\]](https://metacpan.org/pod/Dist::Zilla::Plugin::Git::Push).

# SUPPORT

Homepage:
[https://github.com/latk/p5-Dist-Zilla-PluginBundle-Author-AMON](https://github.com/latk/p5-Dist-Zilla-PluginBundle-Author-AMON)

Bugtracker:
[https://github.com/latk/p5-Dist-Zilla-PluginBundle-Author-AMON/issues](https://github.com/latk/p5-Dist-Zilla-PluginBundle-Author-AMON/issues)

# AUTHOR

amon - Lukas Atkinson (cpan: AMON) <amon@cpan.org>

# COPYRIGHT

Copyright 2017 Lukas Atkinson

This library is free software and may be distributed under the same terms as perl itself. See [http://dev.perl.org/licenses/](http://dev.perl.org/licenses/).
