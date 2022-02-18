# Reproducing duplicated `DepsErrors` caused by executable imports

This repository contains a minimal module in which `go list` emits duplicated `DepsErrors` entries, that in turn cause `gopls` to error out with

```text
internal error: go list gives conflicting information for package <name> [<name>.test]
```

As far as I can tell, this looks very much like [golang/go#34321].

Besides the errors emitted by `go list` and `gopls`, a package like this compiles, tests and runs (if `main`) just fine.


## A confluence of factors

For this issue to manifest, a package has to have two things:

1. a Go file that imports executable packages (such as tools)
2. a Go test file

One without the other does not make `go list` behave this way.


## Seeing is believing

To observe the `DepsErrors` duplication, run

```shell
go list -mod=readonly -e -json -test=true -tags=tools ./foo | grep -B3 -A7 ImportStack
```

```text
	],
	"DepsErrors": [
		{
			"ImportStack": [
				"github.com/antichris/go-repro-tools-cause-duplicated-depserrors/foo"
			],
			"Pos": "foo/tools.go:5:8",
			"Err": "import \"rsc.io/hello\" is a program, not an importable package"
		}
	],
	"TestGoFiles": [
--
	],
	"DepsErrors": [
		{
			"ImportStack": [
				"github.com/antichris/go-repro-tools-cause-duplicated-depserrors/foo"
			],
			"Pos": "foo/tools.go:5:8",
			"Err": "import \"rsc.io/hello\" is a program, not an importable package"
		}
	]
}
--
	],
	"DepsErrors": [
		{
			"ImportStack": [
				"github.com/antichris/go-repro-tools-cause-duplicated-depserrors/foo"
			],
			"Pos": "foo/tools.go:5:8",
			"Err": "import \"rsc.io/hello\" is a program, not an importable package"
		},
		{
			"ImportStack": [
				"github.com/antichris/go-repro-tools-cause-duplicated-depserrors/foo"
			],
			"Pos": "foo/tools.go:5:8",
			"Err": "import \"rsc.io/hello\" is a program, not an importable package"
		}
	],
	"TestGoFiles": [
```

The last two matches are two identical entries in the same `DepsErrors` block. Although they don't appear to be contradictory to the human eye, `gopls` errors out with a complaint about "conflicting information".

[golang/go#34321]: https://github.com/golang/go/issues/34321
	(cmd/go: 'go list -test' prints main package twice · Issue #34321 · golang/go)
