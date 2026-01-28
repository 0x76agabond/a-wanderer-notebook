# **Modular Smart Contract Architecture**

## **1. What Is Modular Design**

Modular design is an architectural approach that organizes a system into independent parts, each owning a clearly defined responsibility.  

Each part can function independently as a standalone application, or be composed together with other parts to form a complete system.

The key idea is that each component defines a clear boundary of responsibility. This organizes the system into distinct logical units that can be developed, reasoned about, and evolved without requiring full knowledge of the entire system.

![Modular Design](https://raw.githubusercontent.com/0x76agabond/a-wanderer-notebook/refs/heads/main/images/modular.png)

### **The Real Goal of Modular Design**

Modular design is often associated with benefits such as maintainability, reusability, scalability, and parallel development.
In practice, these are outcomes rather than the primary goal.

The core goal of modular design is how a system is organized, and how developer attention is directed while working on it.

By structuring a system into meaningful, well-defined modules:

- Code is divided into smaller, purposeful units that are easier to implement and test.

- Each module can be treated as an independent unit, giving developers more flexibility in how they organize work, code, and collaboration.

- Well-isolated and well-organized components help reduce overall complexity and allow the system to grow in a more natural way over time.

### **Modular Design as Native Separation of Concerns**

Modular design is the native, deployable implementation of Separation of Concerns.  
Boundaries are enforced by architecture rather than convention, allowing concerns to be isolated early before they become entangled.

At its core, whether we call it Separation of Concerns or Modular Design, it is not a strict set of rules.  
It is a programming philosophy about how work and responsibility are organized, rather than a checklist of patterns to follow.

By applying these principles, both teams and individual developers can organize their projects in a clearer, more understandable way, and prevent systems from becoming increasingly messy over time.

### Why Modular Design doesn't slow you down

For projects or teams where speed is the top priority, upfront planning often looks like over-engineering and can feel like it slows things down.

**In practice, this is usually not a real problem.**  
One of the most effective ways to build a modular system is to start with a monolith.  
In the early stages, this allows teams to move fast while the problem space is still unclear.

As the system grows, natural boundaries will start to appear.  
That is the right moment to start grouping logic, defining modules, and introducing structure deliberately.

The trick is to choose an architecture that can natively split and scale when needed, so the project can evolve from a monolith into a modular system without the pain of a full refactor.

Even in cases where a project does not need to change much, organizing code into smaller, well-defined, and easy-to-test blocks can still improve the overall quality of the system and reduce the cognitive load on developers.

### Modular Design Helps Organize Work More Effectively

One of the most frustrating things about starting a new project is not knowing how the work should be organized.

When teams already have experience or existing references, this is manageable.
But when the project is new or no solution clearly fits, things can quickly become messy.

This often leads to building new features while reorganizing existing code, which fragments focus and introduces unnecessary overhead.

Under time pressure, this can push developers to accept partial, patchwork solutions, which accumulate technical debt over time.

**Modular design helps developers in several ways.**

- It reduces the amount of context a developer needs to keep in mind at any given time.

- Each module can be treated as a small application on its own, which makes it easier to reason about, test, and maintain.

- Well-defined modules can often be reused across different projects instead of being rewritten from scratch.  
→ **This creates a natural foundation for shared libraries and reusable modules.**

**Modular design helps teams stay organized even when the solution is not yet clear.**

- Well-understood features can be implemented quickly by familiar team members.

- New or uncertain features can be explored independently by others.

- Some features can be outsourced when time is critical without destabilizing the system.

By isolating concerns early, modular design allows teams to keep moving forward despite uncertainty.

### **Conclusion**

Modular design is a design philosophy that brings many practical benefits.  
With the right architecture and working strategies, it allows a project to start fast and easily, while maintaining good organization and leaving room to grow over time.

By structuring work around clear concerns, modular design reduces how much developers need to keep in their head at any given moment and adds flexibility for both solo developers and teams.

While modular design introduces some planning overhead, in practice this cost is manageable through the right architecture and development approach.

## **2. Modular design for Smart Contracts**

When people talk about modular smart contracts, most of us immediately think about composability and upgradeability.  
Those aspects matter, but modular design goes beyond that.

**At its core, a good modular system is:**

- A system where each concern is fully isolated, with each module able to function as an independent unit, similar to a standalone application.

- A system that can add, remove or update logic without affecting unrelated functions or corrupting state.

- A system that allows developers to work and commit changes confidently,
while remaining isolated from unrelated parts of the system.  

- A system where multiple developers or teams can collaborate asynchronously by working on separate concerns.

- A system that does not rely on a single individual’s knowledge, but is structured to remain approachable and maintainable for whoever is working on it.

### **From Web2 Principles to Web3 Reality**

In Web2 systems, these goals are commonly achieved through `Clean Code` and principles like `SOLID`.

In Web3, directly applying SOLID becomes impractical due to gas costs, EVM constraints, and immutability.

However, things are not as bad as they first appear. Web3 also provides some structural advantages that Web2 does not have.

- The event and log system is native to the EVM and behaves consistently across EVM-compatible chains. Logs and events are first-class citizens, making event-driven architectures a natural fit for Web3 systems.

- Layer 2 solutions significantly reduce gas costs, making more structured and modular designs economically viable compared to early Layer 1–only environments.

While the mechanics are different, the ideas behind those principles still matter.  
Instead of forcing Web2 patterns onto Web3, a better approach is to forget the dogma and cherry-pick what we actually need.

### **Reframing Modular Design in Smart Contracts**

If we cherry-pick the core needs of modular design and try to adapt them to the realities and constraints of the EVM, we quickly end up with a lot of different directions.

Since there is no single obvious answer, we keep things simple and look for solutions that cover the most requirements.

From that point of view, some solutions start to make more sense. One of them is:

**ERC-2535 — The Diamond Standard**

Diamond addresses these needs at both the function and storage level.  
By operating on function selectors and enforcing clear boundaries between logic and storage, it provides a practical foundation for modular smart contract systems under EVM constraints.

![Diamond](https://raw.githubusercontent.com/0x76agabond/a-wanderer-notebook/refs/heads/main/images/diamond.png)

To put theory into practice, let’s look at how Diamond separates state from logic through a decoupled storage pattern.

**Storage example**

```solidity
bytes32 constant STORAGE_POSITION = keccak256("storage.identifier");

struct Storage {
    uint256 value;
}

function storage_() internal pure returns (Storage storage s) {
    bytes32 position = STORAGE_POSITION;
    assembly {
        s.slot := position
    }
}
```

**Facet example**

```solidity
contract Facet {
    function setValue(uint256 _value) external {
        Storage storage s = storage_();
        s.value = _value;
    }

    function getValue() external view returns (uint256) {
        Storage storage s = storage_();
        return s.value;
    }
}
```

**In this model**

- Storage does not need to know which logic interacts with it.  
It defines the state structure and provides a deterministic access point to that state.

- Facet and selectors does not need to understand how storage is globally organized.   
It only interacts with the specific state required to perform its behavior.

You may worry about keccak collisions, but even with quantum techniques, existing attacks only apply to reduced-round variants and are not a practical concern for the 24-rounds keccak used in the EVM, as discussed [here](https://link.springer.com/chapter/10.1007/978-3-031-22969-5_22).

In addition, Solidity also relies on keccak internally to derive storage locations for mappings and dynamic arrays, as described in the [language documentation](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#mappings-and-dynamic-arrays).

### **What Is New with Diamonds in 2026?**

When the Diamond Standard was first introduced, its primary goal was to overcome the 24KB contract size limit.  
At the time, it quickly became one of the most flexible ways to implement large and upgradeable smart contract systems on-chain.

As the ecosystem evolved and real-world usage increased, it started to feel like the Diamond Standard had more to offer than its well-known strengths, especially when viewed as a way to structure a project.

Its natural separation of logic and storage turned out to be a strong foundation for modular design.
This structure supports incremental development, parallel work, and clearer ownership of responsibilities as systems grow.

Diamond represent a different approach that provides a strong foundation for projects to grow, evolving from simple setups into larger systems.

Here are a few efforts from the Diamond community. Let’s see what each one offers:

### ERC-8109 (Diamonds, Simplified)
*A simplified Diamond architecture for modular smart contract systems.*

ERC-8109 reduces the cognitive load of working with Diamonds by clarifying how system changes and contract visibility are supposed to work.

**It helps align expectations between developers, tools, and auditors by clearly defining things like:**

- How explorers, indexers, and tooling should interact with Diamonds.

- How upgrade and introspection behavior is expected to work.

On top of that, ERC-8109 explicitly documents several important behaviors that are easy to miss in real-world implementations, such as ownership, shared and composed facets, immutable Diamonds or immutable functions,  etc.

Overall, it provides the necessary information to build a clearer mental model.   
Because of that, getting familiar with ERC-8109 is the recommended starting point before diving deeper.

### ERC-8110: Domain Architecture for Diamonds
*An architectural pattern that organizes Diamond Storage by domain using ERC-8042 identifiers.*

![8110_Design](https://raw.githubusercontent.com/0x76agabond/a-wanderer-notebook/refs/heads/main/images/diamond8110.png)

ERC-8110 introduces a new way to organize code and manage storage in Diamond-based systems, built around the idea of domains.

**At a high level, the idea is quite simple**:

- Each function selector is meant to solve a specific concern in the real world.

- A group of related concern-solvers, when combined, forms a complete feature, which is referred to as a domain.

- Each domain has clear boundaries and can be treated as a small, self-contained application within the larger system.

In other words, instead of organizing a Diamond primarily around facets, ERC-8110 encourages structuring the system around the relationship between function selectors and domains.

**So what does this actually give us?**

- It isolates storage at the architectural level, making storage collisions conceptually impossible except through plain human error.

- It decouples logic from state management. Once storage is organized by domain, upgrades become more predictable and follow a clear structure instead of relying on fragile conventions.

- It removes storage responsibility from facets. Facets become pure units of behavior, free to change without worrying about long-term state layout.

As core concept, ERC-8110 shifts the focus from “how do we avoid breaking storage?” to “how do we evolve the system safely”. 

**Consequences of Isolation**

Once storage is properly isolated at the architectural level, a number of practical benefits naturally start to emerge.

- Lower organizational overhead. A developer can come back months or even years later to add new features without worrying about accidentally affecting other domains.

- Reduced cognitive load. Developers only need to understand and focus on the domain they are working on, instead of keeping the entire system in their head.

- Better support for collaboration. Multiple teams can work on different domains in parallel without stepping on each other’s toes.

In practice, this kind of isolation and organization makes Diamond systems easier to evolve over time, even as teams, requirements, and priorities change.

### Compose  
*A smart contract library that helps developers build modular smart contract systems.*

<img src="https://raw.githubusercontent.com/0x76agabond/a-wanderer-notebook/refs/heads/main/images/Compose.jpg" width="500">

Compose represents a new generation of Diamond tooling, built around a **Smart Contract Oriented Programming (SCOP)** approach.

At its core, Compose provides three core capabilities.

**1. On-chain Predeployed Facets (Not Ready Yet)**

Compose is still at an early stage, and this capability is not available yet.  
However, the idea is central to the long-term direction of the project and too valuable to ignore, so it is worth calling out explicitly.

- Compose plans to offer a collection of immutable, audited facets deployed on-chain at deterministic addresses, such as `ERC20TransferFacet`, `ERC721TransferFacet`, etc.

- With this approach, a project would only need to deploy a Diamond proxy and register the required function selectors to start using the functionality immediately.

This direction aims to significantly reduce deployment effort and setup costs, especially for widely used standards.

**2. Modules Compatible with On-chain Facets**

Diamonds are flexible by design, but relying only on predeployed facets would lock projects into predefined behavior.

- To avoid this, Compose provides modules that are fully compatible with the storage layout of the on-chain facets. These modules allow projects to extend, customize, or build additional logic on top of the existing facets.

- Because modules and on-chain facets share compatible storage layouts, developers can create custom features without being constrained by typical no-code or low-code limitations.

**3. Dedicated Facet Implementations**

- For projects that, for any reason, cannot or do not want to use predeployed on-chain facets, Compose also provides standalone facet implementations.

- Developers can deploy and use their own versions of facets, such as an `ERC20Facet`, without relying on Compose’s predeployed contracts, while still following the same architecture and compatibility rules.

**A Very Different Way to Approach Dapps**

At first glance, Compose can look like a library built for Diamonds in the same way npm works for JavaScript.
However, when you look at it more closely from an EVM perspective, it reveals a set of advantages that are quite different, and in some cases, unique.

- Users only deploy a proxy contract, which creates an effect similar to `ERC-1167 Minimal Proxy`. This significantly reduces deployment gas costs. At the same time, Diamonds retain their natural flexibility and upgrade paths through modules, rather than sacrificing extensibility for simplicity.

- Using predefined, audited facets means relying on contracts that are battle tested, immutable, and already audited. This greatly reduces supply chain risk and shortens the path from idea to a production ready system in a way that is hard to achieve with traditional smart contract development.

- An independent developer, working within a limited timeframe and writing relatively little custom code, can still deliver a system that meets industry-level standards.

Taken together, Compose doesn’t just optimize deployment or reuse code.
It changes how smart contracts can be assembled, evolved, and shipped in practice.

## **3. Diamonds as a Development Framework**

At this point, we can probably agree on one thing. **Diamonds give us a solid foundation for modular smart contract design.**

We also have a growing set of standards and tools that make working with Diamonds easier and more practical.  
Each improvement helps in a different way, but which ones make sense depends on the project and its constraints.

So instead of adopting everything at once, we’ll go through these improvements together, discuss the trade-offs, and see when it actually makes sense.

### **Before we start**

Before diving deeper, it’s worth setting a clear scope for this section.

- We’ll treat ERC-8109 as the default starting point for new Diamond projects. It offers a clearer and more transparent model, along with a reasonable migration path from ERC-2535. For this discussion, we’ll assume we are early adopters and build our foundation on top of it.

- Since this discussion focuses on Diamonds, we’ll avoid comparisons with other patterns. Readers are still encouraged to think about alternatives and ask themselves how the same problems might look if approached with a different pattern.

Let's learn together.

### **For small and fast-moving projects**

First, take a step back and look at your project.

If your project is performance critical, has very few functions, and you are sure it will never need upgrades.  
→ Use a monolithic immutable contract. Diamond is not a good fit for this case.  

You may still consider a hybrid approach: 
- Keep the core logic in an immutable contract, and use Diamond only as an extension layer via hooks.  
- This can reduce the number of contracts you need to manage, but it is a very project-specific decision.

If your project needs to ship fast with a limited budget, check whether the features you need are already supported by Compose.  
→ If they are, do a quick internal review of the relevant predeployed facets.  
→ If they fit your needs, jackpot! Just deploy the Diamond, add selectors from predeployed facets, write some tests, and go grab a coffee.

If your project needs to ship fast and is supported by Compose, but security or business requirements prevent you from using predeployed facets, meaning you must deploy everything yourself, then the decision depends on what you value.

If you or your team care about code readability, maintainability, a clean architecture, and long term sustainability.  
→ Treat Diamond as an investment. Start with ERC-8109, audit the Compose features you plan to use, and build from there.

What if Compose does not support your feature yet?  
If you want to, you can open an issue on their GitHub. It’s worth a try.  
Compose is still evolving, and many features start from real project needs.  

If you’re willing to handle it yourself, this is where architectural tricks come into play. Start simple with a monolith and improve over time.  
→ Use Diamond with single storage and one implementation facet, then scale gradually as the project grows.  

Since the storage and logic here are application-specific, you may consider applying ERC-8110 and using sub-domains.

**Sub-domain example**  
```solidity
/// @custom:storage-location erc8042:org.project.system.domain.v1
bytes32 constant DOMAIN_STORAGE_POSITION = keccak256("org.project.system.domain.v1");

struct DomainStorage {
    uint256 uint_256;
    mapping(uint8 => uint256) mapping_256;
}

/// @custom:storage-location erc8042:org.project.system.domain.v1.subdomain
bytes32 constant SUB_DOMAIN_STORAGE_POSITION = keccak256("org.project.system.domain.v1.subdomain");

struct SubDomainStorage {
    uint32 uint_32_1;
    uint32 uint_32_2;
    uint32 uint_32_3;
}
```

With traditional storage layouts, appending new variables can easily break packed slots.  
By introducing a sub-domain, you gain an escape hatch.

- Variables that pack well can be appended to the sub-domain.

- Variables that would break packing can go into the main domain.

→ This allows the layout to naturally optimize itself over time.  
→ This also frees you from the burden of having to get everything exactly right from the beginning.

As the system grows and domains become clearer, you can introduce new domains and facets later, when they actually make sense.

Setting up this structure early also makes the project easier to read and reason about for developers. More importantly, it allows the system to keep improving both during development and after deployment, without forcing painful trade-offs or refactors.

### **For long term projects**

This is where Diamonds shine.  
Their nature fits projects that are expected to change over time, while remaining readable, manageable, and easy to audit.  

Instead of continuing to repeat what we already know, we'll explore how long-term projects can actually be managed using the newer improvements discussed above.

### What we already have

As discussed earlier in **For small and fast-moving projects**, the Diamond ecosystem is active and continues to mature, with Compose representing a practical step in that direction.

The advantages that Compose brings to long-term projects are the same as those it offers to smaller ones.  
Even in cases where you are not allowed to use predeployed facets or external libraries, Compose remains useful as a reference.

Its codebase can serve as both practical documentation and architectural guidance.  
This reduces the effort required compared to studying monolithic contracts and then migrating them into a Diamond-based system.

Given these building blocks, the real question becomes how they fit into the day-to-day development of a long-term project.

### Incremental Development

One of the biggest challenges in long-term projects is being forced to think about planning from the very beginning.

- If the problem your project is solving already has a known solution, this is still manageable.

- But if the problem is entirely new, or the business logic is complex, planning and exploration alone can easily consume more than half of the total project timeline.

This becomes even more difficult when you are working solo.  
The amount of work and knowledge required to reach a usable product can demand a significant time investment, sometimes to the point of feeling almost impossible.

Beyond purely technical concerns, you also have to deal with business problems, which often means learning an entirely new domain from scratch.

The flexibility that comes from combining ERC-8109 and ERC-8110 forms a solid foundation for handling these kinds of challenges.

Instead of trying to finalize the entire plan upfront, the work can be broken down and accumulated gradually over time.

### A solo developer’s story

Now that we see the problem, let’s look at how we can deal with it using Diamonds.

- In practice, a solo developer may start by building a list of features, something AI can already assist with reasonably well, and exploring how those features relate to each other.

- Next, we set up the external interaction layer, which is already handled well by ERC-8109. We then choose one feature as a domain and begin implementing the first facet.

- Since ERC-8110 only cares about the relationship between domains and selectors, at an early stage it is reasonable to work with a single facet.

- As concerns become clearer, additional domains can be introduced and facets refactored around those new boundaries.

With each completed domain, we gain a concrete feature to review and refine our understanding of the system.  
Each domain can be treated as a small product on its own, which can be tested and evaluated independently.

From time to time, storage updates or code refactors are still required. However, when using ERC-8110 within a disciplined architecture, Diamond upgrades remain highly flexible, and these changes rarely become a bottleneck for development.

### Team collaboration

Many of the problems teams face are similar to those of solo developers. In practice, they are often even more complex.

Without a consistent architecture, work cannot be divided cleanly, and developers inevitably start stepping on each other’s toes. This is the first thing that must be managed.

However, once a structure is in place, collaboration becomes a huge advantage. It allows teams to think together, challenge assumptions, and explore ideas collectively.

This dynamic usually makes the planning phase much lighter compared to working solo. Project concerns tend to surface early, and after a few discussions, a set of well-understood domains often begins to emerge.

At a high level, the idea is simple — split the codebase so that each developer owns a well-defined part, and then connect those parts together.  
With smart contracts, there are multiple ways to do this, and each comes with trade-offs.

- Inheritance is a well-known approach.  
  → If a bug appears in a critical root layer of the inheritance tree, it can easily cascade and compromise the entire system.

- External calls are safe and explicit.  
  → However, they introduce higher gas costs and tend to fragment data across contracts.

- Delegatecall is conceptually powerful.  
  → It comes with strict requirements around storage management, which can be difficult to maintain at scale.

**Diamond provides an architecture that handles these trade-offs more effectively.**

- When domains are identified early, work can be divided naturally around domains and sub-domains.

- Because each domain or sub-domain manages its own storage, a bug is naturally contained within that domain and the specific flows that interact with it.   
  → Unrelated parts of the system remain unaffected.

- The smaller the domains, the smaller the scope of impact when something goes wrong.  
  → However, this comes with higher organizational overhead, as the number of directories and files increases.  
  → This is a trade-off teams should consider consciously.

- This structure enables flexible working models.  
  → Well-understood functions or facets can be implemented quickly to deliver features.  
  → Domains that require deeper research can be isolated and explored independently, with their own timelines.  

- When time or budget constraints apply, domains can be clearly defined, documented, and offloaded to external contributors without destabilizing the rest of the system.

After all, code that is split into simple, isolated blocks of logic is easier to read and understand, saves time when collaborating with others, and naturally becomes easier to audit.  

For solo developers, this also works as a natural reminder when coming back later.  

### **Immutable Diamonds**

Diamonds are often seen as an architecture designed for upgradeability and flexibility.  
In practice, however, Diamonds can also be made immutable.

This is an interesting concept on its own.   
→ We can use the ability to update in order to give up the ability to update.

Here is how it works:

- Deploy the Diamond and initialize it with all required selectors, while omitting any selectors that enable upgrade or diamond cut operations.  
  The result is an immutable Diamond from the start.

- Develop the system incrementally with upgrade support, then remove or disable all upgrade or diamond cut selectors once the design has stabilized.  
  The result is an immutable Diamond at launch.

- Keep the upgrade mechanism in place and continue operating the system as upgradeable when long-term evolution is required.

In all cases, the development process benefits from the same modular structure and separation of concerns.  
The choice between immutable and upgradeable deployment becomes a finalization decision, rather than a constraint imposed early in development.

There is an interesting observation behind this approach:

- We can decide when to introduce immutability at any point in the project lifecycle.

- We can achieve immutability without adding any additional code.

- We can clearly separate the concepts of ownership and upgradeability.

In practice, this resolves one of the most common tensions in smart contract development by allowing us to move fast and stay flexible during development while still delivering a trustworthy and immutable system at launch.


