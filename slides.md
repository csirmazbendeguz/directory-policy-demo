---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Directory - Content Policies
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 35min
---

# Directory
## Content Policies

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

#

<img src="/directory-record-ingestion.png" alt="How a record enters Directory" class="mt-2 mx-auto max-h-114 rounded" />

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: slide-up
level: 2
---

# How a record enters Directory

- 📤 **Push** — a client stores a record with `dirctl push`
- 📦 **Import** — `dirctl import` turns an MCP or A2A registry entry into a record, then pushes it
- 🔄 **Sync** — `dirctl sync` copies another Directory's OCI registry
- ⚡ **Autosync** — Directory pulls a record announced on the DHT by an allow-listed peer

## How to filter out records we don't want?

---

# How to filter out records?

<img src="/badge-policy.png" alt="Records pass through the Policy Engine and split" class="mt-2 mx-auto max-h-104 rounded" />

---

# Content Policies

<img src="/policy-engine-hand-stamp.png" alt="Records pass through the Policy Engine and split" class="mt-2 mx-auto max-h-104 rounded" />

---

# Policies

A (content) policy describes which records are allowed in Directory

It's made of:
- 📋 **1. Evaluations** - evaluates records & creates verdicts
- ⚖️ **2. Verdicts** - stores evaluation results 
- ⚙️ **3. Operations** - allow/reject an operation (may use verdicts)

---

#

<img src="/policy-flow.png" alt="Evaluations create verdicts, which operations use to allow or reject" class="mt-4 mx-auto max-h-100 rounded" /> 

---

# Operations

- 📥 **pull** - allow `dirctl pull` operation
- 🔍 **search** - filter `dirctl search` results
- 📤 **push** - allow `dirctl push` operation
- ⚡ **autosync** - allow a record pulled in from a peer
- 🗂️ **index** - allow a record when it is indexed

---

# pull config

```yaml
server:
  policy:
    evaluations:
      - provider: cel
        name: signed
        expressions:
          - 'meta.signature.status == SignatureStatus.SIGNATURE_STATUS_VERIFIED'
    operations:
      - provider: cel
        op: ["pull"]
        name: serve-signed-records
        expressions:
          - 'verdicts["cel:signed"].compliant_and_current'
```

---

<SlidevVideo controls class="mt-4 mx-auto max-h-72">
  <source src="/pull-operation.mp4" type="video/mp4" />
</SlidevVideo>

---

# search config

```yaml
server:
  policy:
    evaluations:
      - provider: cel
        name: signed
        expressions:
          - 'meta.signature.status == SignatureStatus.SIGNATURE_STATUS_VERIFIED'
    operations:
      - provider: cel
        op: ["search"]
        name: serve-signed-records
        expressions:
          - 'verdicts["cel:signed"].compliant_and_current'
```

---

<SlidevVideo controls class="mt-4 mx-auto max-h-72">
  <source src="/search-operation.mp4" type="video/mp4" />
</SlidevVideo>

---

# push config

```yaml
server:
  policy:
    operations:
      - provider: cel
        op: ["push"]
        name: cisco-authored
        expressions:
          - 'has(record.authors)'
          - 'record.authors.exists(a, a == "Cisco Systems Inc.")'
```

---

<SlidevVideo controls class="mt-4 mx-auto max-h-72">
  <source src="/push-operation.mp4" type="video/mp4" />
</SlidevVideo>

---

# default config

```yaml
server:
  policy:
    operations:
      - provider: oasf
        op: ["push", "autosync", "index"]
        name: schema
        schema_url: "https://schema.oasf.outshift.com"
```

## Only accepts valid OASF records

---

# Use-case: filter by signer identity

Only return records by trusted signers

```yaml
server:
  policy:
    operations:
      - provider: cel
        op: ["pull", "search"]
        name: serve-cisco-signed
        expressions:
          - 'verdicts["cel:signed-by-cisco-email"].compliant_and_current'
    evaluations:
      - provider: cel
        name: signed-by-cisco-email
        expressions:
          - 'meta.signature.status == SignatureStatus.SIGNATURE_STATUS_VERIFIED'
          - 'meta.signature.signer_identity == "bcsirmaz@cisco.com"'
```

---

# Use-case: before sync

<img src="/sync-before.png" alt="Evaluations create verdicts, which operations use to allow or reject" class="mt-4 mx-auto max-h-100 rounded" /> 

---

# Use-case: after sync

<img src="/sync-after.png" alt="Evaluations create verdicts, which operations use to allow or reject" class="mt-4 mx-auto max-h-100 rounded" /> 

---

<SlidevVideo controls class="mt-4 mx-auto max-h-72">
  <source src="/sync-use-case.mp4" type="video/mp4" />
</SlidevVideo>
