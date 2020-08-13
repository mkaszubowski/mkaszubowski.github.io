---
layout: post
title:  "How to regognize good boundaries in modular software design"
date:   2020-08-13 08:30:00 +0100
tags:  development modular-design
skip_related: true
---

Is the boundary for your module* right? Is the module coherent and loosely
coupled with the rest of the system? Here are some things to look at to assess
your design:

{: .tip}
\* you can replace the word "module" with "Bounded Context" or
"(micro)service"

- How easy it is to modify the module (its implementation, internal data
  structures, data storage, business rules) without affecting the rest of the
  system?
- How easy it is to change the integration with other modules. If we
  use/communicate with X from our module, could we easily replace X with Y
  (providing similar functionality)?
- Would it be easy to replace the module with off-the-shelf software without
  affecting the rest of the system?
- How easy is it to remove the module when the business need is no longer there?
- Are changes in the interface connected with required behaviour change, or do
  they result from changing implementation of one of the modules?
- Is the module easy to test in isolation? Is it necessary to do a big setup to
  create entities from other modules/contexts?
- Are you able to understand what the module does without looking at the
  implementation of its dependencies?
- Are you able to describe the purpose of the module in one simple sentence?

## Interested in modular design?
- [Decomposing domain models based on
  lifecycles](/2020/06/24/ecomposing-models-lifecycle.html) shows an example
  software decomposition, which can be applied to many real-life domains.
- [The benefits of modular software
  design](/2020/06/02/modular-software-design-benefits.html) and [The costs of
  modular software design](/2020/05/28/costs-of-modular-software-design.html)
  will help you decide whether more modular software is right for you.
