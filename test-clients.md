---
title: Test clients
layout: home
nav_order: 8
---

# Test clients

We have three available open clients, all of which allow arbitrary redirect URIs.
MitID requires MitID PP test users, which can be created and managed in MitID's Test Tool.

## PP environment open clients

> Authority: https://pp.netseidbroker.dk/op

### Code flow client

| Parameter     | Value                                                                                    |
|---------------|------------------------------------------------------------------------------------------|
| client_id     | 0a775a87-878c-4b83-abe3-ee29c720c3e7                                                     |
| client_secret | rnlguc7CM/wmGSti4KCgCkWBQnfslYr0lMDZeIFsCJweROTROy2ajEigEaPQFl76Py6AVWnhYofl/0oiSAgdtg== |
| scope         | openid mitid                                                                             |
| response_type | code                                                                                     |
| redirect_uri  | ANY ALLOWED                                                                              |

### Hybrid flow client

| Parameter     | Value                                                                                    |
|---------------|------------------------------------------------------------------------------------------|
| client_id     | c0beb4dc-69d1-4316-8167-2d0a62816103                                                     |
| client_secret | HrlMPtMS+Xu0CtsMeTmvwVPx/tETSk/JZXzSo42tpMMI6gyx7G8jSKhEiNccz4HAfV69tGttPDwxhK5fUntwOA== |
| scope         | openid mitid                                                                             |
| response_type | id_token code                                                                            |
| redirect_uri  | ANY ALLOWED                                                                              |

### Implicit flow client

| Parameter     | Value                                                                                    |
|---------------|------------------------------------------------------------------------------------------|
| client_id     | 93ed8e0d-93ad-405c-b1ac-8bf13d484941                                                     |
| client_secret | AfvRfDFt95kLQ0L10oh9AiJJyyFNHd/0x7IJAGdLMAQrSQPpkp7tSmuVVKtAjP5Oog7WIEnrMu+2WjRbT0A9cg== |
| scope         | openid mitid                                                                             |
| response_type | id_token                                                                                 |
| redirect_uri  | ANY ALLOWED                                                                              |
