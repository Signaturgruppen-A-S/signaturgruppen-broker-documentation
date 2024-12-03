---
title: Pushed Authorization Request (PAR)
layout: home
parent: Transaction signing
nav_order: 5
---

# Transasction signing with Pushed Authorization Request (PAR)

## Signtext API integration and PAR
As an alternative to the Signtext API integration utilized to generate **signtext_id** parameters for the OIDC flows, the PAR endpoint available at Signaturgruppen Broker supports setting up the signtext payload directly through a single PAR integration. 

The PAR endpoint support the following additional two parameters, which must both be specified to be applied to the flow

<table>
   <tbody>
      <tr>
         <th>
            <p><strong>PAR parameter</strong></p>
         </th>
         <th>
            <p><strong>Description</strong></p>
         </th>
      </tr>
      <tr>
         <td>
            <p>signtext_base64</p>
         </td>
         <td>
            <p>Signtext payload in Base64 encoding</p>
         </td>
      </tr>
     <tr>
         <td>
            <p>signtext_type</p>
         </td>
         <td>
            <p>One of <b>text</b>, <b>html</b> or <b>pdf</b></p>
         </td>
      </tr>
   </tbody>
</table>

When both parameters have been specified, the resulting OIDC flow is equivalent to setting up the **signtext_id** parameter directly using the Signtext API. 
This allows for integrations utilizing the transasction signing flow without the need for two API calls and an active integration to the Signtext API.
