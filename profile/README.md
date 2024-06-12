### Engineering Tenets

_[Unless you know better ones:](https://aws.amazon.com/blogs/enterprise-strategy/tenets-supercharging-decision-making/)_

- **We are full-stack, fungible engineers**: we have a good grasp and understanding of UX best practices and patterns, distributed web architecture and the full-stack life cycle of a request. 
- **Implementation is communication**: code is written to communicate intent, optimized to be maintained and modified by others. Tests support our development process, not the other way around. We test the behaviour of our code, not the implementation.
- **Optimize for what we know**: principles are exercised pragmatically, and while everything is a tradeoff, not everything is a debate. We record decisions and maintain a repository of knowledge, not procedures. 
- **Keep it simple and fast**: we do not need an issue, ticket, or user story for every pull request. It is perfectly fine (and desired) to make cosmetic changes, small improvements, minor enhancements in a pull request, but we always make sure to document the intent of the change somewhere.

## Engineering SOP

See [Engineering Profile](https://github.com/dittodev/.github/wiki/Engineering-profile) for more.

#### Code reviews/Code changes/Tracking

We use Github pull requesting and encourage keeping the history clean.
Most of this will be configured on Github so no need to document here.

Common wisdom: keep changes small and topical, avoid many concurrent
pull requests open, prioritize reviewing over writing more code (and
avoid review marathons).

You do not need an issue, ticket, or user story for every pull request.
It is perfectly fine to make cosmetic changes, small improvements, minor
enhancements in a pull request, but always make sure we document the
intent of the change, not the change itself.

#### Architecture Decision Records (ADR)

Document small architectural decisions using [architectural design
records](https://adr.github.io/). ADRs are maintained alongside the code
that they represent.

ADR scope: any decision related to code that fits a single page of text.
Large scope like choice of some framework or data storage layer do not
fit this scope and should be taken elsewhere.

#### Engineering Design Documents (EDD)

Create an engineering design document for new endpoints (services) or
changes to existing endpoints, changes to existing infrastructure or
major (software) component designs.

EDDs must be optimized to communicate the most significant changes
regarding security, latency, and additional technologies. The EDD
describes the system as we intend to build it, includes a diagram at C2
level and when needed a request-response-diagram or process flow
diagram. An EDD is not a research document or decision document.

#### Rollback, revert, fix

Never forward fix, and always roll back changes first. Rolling back and
reverting a breaking change unblocks other changes being delivered and
mitigates any additional risk from a forward fix. When in doubt, roll
back and revert. Reverts and rollbacks must be documented in a ticket,
notifying the original author.

## Docs

* [Engineering Profile](https://github.com/dittodev/.github/wiki/Engineering-profile)
* [Engineering sync log](https://github.com/dittodev/.github/wiki/Engineering-sync-log)
* [Engineering design documents (EDD)](https://github.com/dittodev/.github/tree/main/EDD)
* [Architecture decision records (ADR)](https://github.com/dittodev/.github/tree/main/ADR)


<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
