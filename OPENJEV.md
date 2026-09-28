# OpenJEV support

This fork of [typesafe.zig](https://github.com/mattneel/typesafe.zig) adds **optional** support for
[OpenJEV](https://openjev.sh), a free community gateway to the same Jev model that
[TypeSafe](https://typesafe.ai) serves. OpenJEV speaks the same `POST /v1/systemone`
request/response contract; only the base URL, model id and API key differ.

**TypeSafe stays the default.** Anyone with a `TYPESAFE_API_KEY` sees zero behaviour change.
OpenJEV is opt-in.

## What was added

- `src/Client.zig`
  - `Provider` enum (`typesafe`, `openjev`) and an `Options.provider` field.
  - `openjev_base_url` (`https://api.openjev.sh`), `openjev_model` (`openjev`),
    `env_openjev_api_key` (`OPENJEV_API_KEY`) and `env_provider` (`JEV_PROVIDER`) constants.
  - `init` and `initFromEnv` resolve the provider and pick the matching base URL, model id and
    key. `providerFromEnv` implements the auto rule.
  - The retry/error mapping already retries every 5xx (including OpenJEV's 503 overload) and 429,
    so no change to `Retry.zig` / `errors.zig` was needed.
- `README.md` — an OpenJEV note after the intro, a `provider` row in the configuration table,
  and an `### OpenJEV` subsection with an example.
- `docs/SPEC.md` — a `provider` row in the configuration table and a short OpenJEV paragraph.

No TypeSafe code path, name, default, header or env var was renamed, removed or re-defaulted.
The `LICENSE`, `CHANGELOG.md` and credits are untouched.

## Provider selection rule

Resolved in this order:

1. Explicit `Options.provider` wins.
2. `JEV_PROVIDER` env var: `openjev` selects OpenJEV, anything else TypeSafe.
3. Auto: TypeSafe when `TYPESAFE_API_KEY` is set (the unchanged default); OpenJEV when only
   `OPENJEV_API_KEY` is set; `error.MissingApiKey` when neither is set.

When OpenJEV is selected, the defaults become `https://api.openjev.sh`, the `openjev` model id
and `OPENJEV_API_KEY`. `TYPESAFE_BASE_URL` and `TYPESAFE_DEFAULT_MODEL` still override them when
set, so a program can point either provider at a custom host.

## How to configure

```sh
# Auto: only the OpenJEV key is set, so OpenJEV is selected.
export OPENJEV_API_KEY=...
```

```zig
// Explicit option
var client: typesafe.Client = try .init(gpa, io, .{
    .provider = .openjev,
    .api_key = openjev_key,
});

// Or force OpenJEV from the environment even when both keys are present
// (JEV_PROVIDER=openjev), then:
var client: typesafe.Client = try .initFromEnv(init.gpa, init.io, init.environ_map, .{});
```

Get an OpenJEV key from <https://openjev.sh/dashboard>.

## How it was verified

- A live `POST https://api.openjev.sh/v1/systemone` request with model `openjev`, state `ping`
  and one `noul` question, authenticated with `OPENJEV_API_KEY`, returned HTTP 200.
- `grep` confirmed no hardcoded `https://api.typesafe.ai` default remains in the OpenJEV path;
  the TypeSafe default is unchanged and still present for the `typesafe` provider.
- The existing `initFromEnv` and `init` unit tests in `src/Client.zig` are unchanged and still
  exercise the TypeSafe default path.

The project's own build/tests were not run (third-party code is never executed during a port).

## Upstream

Original project: <https://github.com/mattneel/typesafe.zig> by [@mattneel](https://github.com/mattneel).
