# 服务治理

当我们在涉及到服务治理的时候，首先需要明确的是，只有事物存在稳定运行风险的时候，那么我们就需要 "治理"。

根据我们对于应用程序的服务了解，如果它承担的业务功能十分简单，运行过程是相对透明，即使出现问题也能较快发现，而后定位并处理，这种情况下，没有必要进行 "治理"。但是当服务承担的业务功能变多，业务规模变大，那服务在维护的过程中，会面对更多的问题，服务治理就变得必要 -- 因为它要将业务所涉及到的很多服务，规整到一个条理上。 当然了，服务治理与软件项目技术架构是息息相关。

![](https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=MmVjNjlmNzU0NjQ0N2FiODNjMDI1NDA3NGYxZDczZWVfYWNWbGFiYWxFNENVOTY0cVFuWlFsUXJ1cjhLWlNsc2RfVG9rZW46V3UxRWIxaEh4b0dUUDR4VU56aWNvVHhubjJmXzE3MTgxODk5NTQ6MTcxODE5MzU1NF9WNA)

## 三种形态

<figure><img src="https://y04h4pmzvxx.feishu.cn/space/api/box/stream/download/asynccode/?code=NzZjMTVmZGQxZTEwOTE5MTljNjljMTkyNmMyYmUzNTJfRXk1VWJWeVVubUM3aVNPV3JqNjF0QWRhZXluZjNyVGZfVG9rZW46SjlMUmJ4ODBNb1YxSlp4N1VDcWN1aFhHblRiXzE3MTgxOTAwNzA6MTcxODE5MzY3MF9WNA" alt=""><figcaption></figcaption></figure>

<table data-view="cards"><thead><tr><th></th><th></th></tr></thead><tbody><tr><td><strong>代码内嵌模式</strong>：</td><td>将服务治理的代码，直接内嵌到核心程序代码逻辑中，由于服务治理代码和应用程序代码紧耦合在一起，在后续管理的时候，尤其是当我们的服务治理的功能规模庞大到一定程度的时候，导致后期无法管理。</td></tr><tr><td><strong>独立代码模式</strong></td><td>将服务治理的代码以 "独立代码" 的方式存在，然后以程序公共库等方式整合到SDK(Software Development Kit)集合中，接着在应用程序开发的时候，将服务治理的功能嵌入到应用程序的逻辑代码中，这种方法虽然仍然是一种 耦合方式，但是重复性的服务治理功能，直接重用，所以效率还是可以的。当然了，这种模式也有其内在的劣势，SDK的语言和程序语言必须一致，后期维护的时候，必须同时维护。</td></tr><tr><td><strong>sidecar模式</strong>： 是目前服务治理使用最多的一种模式。</td><td>将服务治理的代码以 "独立程序" 的方式存在，以sidecar的方式，运行到业务代码的前端，这样的话，所有业务程序的流量，都可以被服务治理程序进行高度的处理，然后将处理后的流量精准的交给后端的应用业务程序。这种方式的好处是，应用程序和服务治理程序以松耦合的方式存在，在程序开发、程序迭代等层面，互不影响。</td></tr></tbody></table>
