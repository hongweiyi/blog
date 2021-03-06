title: 分布式文件系统元数据服务器模型
tags:
  - 分布式
id: 163
categories:
  - 技术分享
date: 2011-10-01 10:21:35
---

随着非结构化数据的爆炸，分布式文件系统进入了发展的黄金时期，从高性能计算到数据中心，从数据共享到互联网应用，已经渗透到数据应用的各方各面。对于大多数分布式文件系统(或集群文件系统，或并行文件系统)而言，通常将元数据与数据两者独立开来，即控制流与数据流进行分离，从而获得更高的系统扩展性和I/O并发性。因而，元数据管理模型显得至关重要，直接影响到系统的扩展性、性能、可靠性和稳定性等。存储系统要具有很高的Scale-Out特性，最大的挑战之一就是记录数据逻辑与物理位置的映像关系即数据元数据，还包括诸如属性和访问权限等信息。特别是对于海量小文件的应用，元数据问题是个非常大的挑战。总体来说，分布式文件系统的元数据管理方式大致可以分为三种模型，即集中式元数据服务模型、分布式元数据服务模型和无元数据服务模型。在学术界和工业界，这三种模型一直存在争议，各有优势和不足之处，实际系统实现中也难分优劣。实际上，设计出一个能够适用各种数据应用负载的通用分布式文件系统，这种想法本来就是不现实的。从这个意义上看，这三种元数据服务模型都有各自存在的理由，至少是在它适用的数据存储应用领域之内。

<!--more-->

**集中式元数据服务模型**

分布式文件系统中，数据和I/O访问负载被分散到多个物理独立的存储和计算节点，从而实现系统的高扩展性和高性能。对于一组文件，如果以文件为单位进行调度，则不同的文件会存储在不同的节点上；或者以Stripe方式进行存储，则一个文件会分成多个部分存放在多个节点。显然，我们面临的一个关键问题就是如何确保对数据进行正确定位和访问，元数据服务正是用来解决这个问题的。元数据服务记录数据逻辑名字与物理信息的映射关系，包含文件访问控制所需要的所有元数据，对文件进行访问时，先向元数据服务请求查询对应的元数据，然后通过获得的元数据进行后续的文件读写等I/O操作。

出于简化系统设计复杂性的考虑，并且由于大量的历史遗留系统等原因，大多数分布式文件系统采用了集中式的元数据服务，如Lustre, PVFS, StorNext, GFS等。集中式元数据服务模型，通常提供一个中央元数据服务器负责元数据的存储和客户端查询请求，它提供统一的文件系统命名空间，并处理名字解析和数据定位等访问控制功能。传统的NAS系统中，I/O数据流需要经过服务器，而分布式文件系统中，I/O数据流不需要经过元数据服务器，由客户端与存储节点直接交互。这个架构上的变革，使得控制流与数据流分离开来，元数据服务器和存储服务器各司其职，系统扩展性和性能上获得了极大的提升。显而易见，集中式元数据服务模型的最大优点就是设计实现简单，本质上相当于设计一个单机应用程序，对外提供网络访问接口即可，如Socket, RPC, HTTP REST或SOAP等。元数据服务设计实现的关键是OPS吞吐量，即单位时间处理的操作数，这对集中式元数据服务模型尤其关键，因为会受到系统Scale-Up方面的限制。为了优化OPS，该模型对CPU、内存、磁盘要求较高，条件允许的情况下尽量使用高性能CPU、大内存和高速磁盘，甚至后端存储可考虑使用高端磁盘阵列或SSD。在软件架构方面设计，应该考虑多进程/线程(池)、异步通信、Cache、事件驱动等实现机制。至于分布式文件系统名字空间的设计实现，请参考“[分布式文件系统名字空间实现研究](http://blog.csdn.net/liuben/article/details/5993604)”一文，这里不再讨论。实际上，集中式元数据服务模型的缺点同样突出，其中两个最为关键的是性能瓶颈和单点故障问题。

性能瓶颈，这种模型下元数据服务器在负载不断增大时将很快成为整个系统性能的瓶颈。根据Amdahl定律，系统性能加速比最终受制于串行部分的比重，这决定了系统使用并行手段所能改进性能的潜力。这里，元数据服务器就是串行的部分，它直接决定着系统的扩展规模和性能。文件元数据的基本特性要求它必须同步地进行维护和更新，任何时候对文件数据或元数据进行操作时，都需要同步更新元数据。例如，文件的访问时间，即使是读操作或列目录都需要对它进行更新。客户端访问分布式文件系统时，都需要先与元数据服务器进行交互，这包括命名空间解析、数据定位、访问控制等，然后才直接与存储节点进行I/O交互。随着系统规模不断扩大，存储节点、磁盘数量、文件数量、客户端数据、文件操作数量等都将急剧增加，而运行元数据服务器的物理服务器性能毕竟终究有限，因此集中式元数据服务器将最终成为性能瓶颈。对于众所周知的LOSF(Lots of Small Files)应用，文件数量众多而且文件很小，通常都是几KB至几十KB的小文件，比如CDN和生命科学DNA数据应用，集中式元数据服务模型的性能瓶颈问题更加严重。LOSF应用主要是大量的元数据操作，元数据服务器一旦出现性能问题，直接导致极低的OPS和I/O吞吐量。目前，以这种模型实现的分布式文件系统都不适合LOSF应用，比如Lustre, PVFS, GFS。

实际上，性能瓶颈问题没有想像中的那么严重，Lustre, StorNext, GFS等在大文件应用下性能极高，StorNext甚至在小文件应该下性能也表现良好。一方面，首先应该尽量避免应用于LOSF，除非对性能要求极低。其次，对于大文件应用，更加强调I/O数据吞吐量，元数据操作所占比例非常小。文件很大时，元数据数量将显著降低，而且系统更多时间是在进行数据传输，元数据服务器压力大幅下降。这种情形下，基本上不存在性能瓶颈问题了。再者，如果出现性能瓶颈问题，在系统可以承载的最大负载前提下，可以对元数据服务器进行性能优化。优化最为直接的方法是升级硬件，比如CPU、内存、存储、网络，摩尔定律目前仍然是有效的。系统级优化通常也是有效的，包括OS裁剪和参数优化，这方面有很大提升空间。元数据服务器设计本身的优化才是最为关键的，它可以帮助用户节约成本、简化维护和管理，优化的方法主要包括数据局部性、Cache、异步I/O等，旨在提高并发性、减少磁盘I/O访问、降低请求处理时间。因此，在非常多的数据应用下，集中式元数据服务器的性能并不是大问题，或者通过性能优化可以解决的。

单点故障(SPOF，Single Point of Failure)，这个问题看上去要比性能瓶颈更加严重。整个系统严重依赖于元数据服务器，一旦出现问题，系统将变得完全不可用，直接导致应用 中断并影响业务连续性。物理服务器所涉及的网络、计算和存储部件以及软件都有可能发生故障，因此单点故障问题潜在的，采用更优的硬件和软件只能降低发生的概率而无法避免。目前，SPOF问题主要是采用HA机制来解决，根据可用性要求的高低，镜像一个或多个元数据服务器(逻辑的或物理的均可)，构成一个元数据服务HA集群。集群中一台作为主元数据服务器，接受和处理来自客户端的请求，并与其他服务器保持同步。当主元数据服务器发生问题时，自动选择一台可用服务器作为新的主服务器，这一过程对上层应用是透明的，不会产生业务中断。HA机制能够解决SPOF问题，但同时增加了成本开销，只有主服务器是活动的，其他服务器均处于非活动状态，对性能提升没有任何帮助。

**分布式元数据服务模型**

自然地有人提出了分布式元数据服务模型，顾名思义就是使用多台服务器构成集群协同为分布式文件系统提供元数据服务，从而消除集中式元数据服务模型的性能瓶颈和单点故障问题。这种模型可以细分为两类，一为全对等模式，即集群中的每个元数据服务器是完全对等的，每个都可以独立对外提供元数据服务，然后集群内部进行元数据同步，保持数据一致性，比如ISILON、LoongStore、CZSS等。另一类为全分布模式，集群中的每个元数据服务器负责部分元数据服务(分区可以重叠)，共同构成完整的元数据服务，比如PanFS, GPFS, Ceph等。分布式元数据服务模型，将负载分散到多台服务器解决了性能瓶颈问题，利用对等的服务器或冗余元数据服务分区解决了单点故障问题。分布式看似非常完善，然而它大大增加了设计实现上的复杂性，同时可能会引入了新的问题，即性能开销和数据一致性问题。

性能开销，分布式系统通常会引由于节点之间的数据同步而引入额外开销，这是因为同步过程中需要使用各种锁和同步机制，以保证数据一致性。如果节点同步问题处理不当，性能开销将对系统扩展性和性能产生较大影响，和集中式元数据模型一样形成性能瓶颈，这就对分布式元数据服务器的设计提出了更高的要求。这种性能开销会抵消一部分采用分布式所带来的性能提升，而且随着元数据服务器数量、文件数量、文件操作、存储系统规模、磁盘数量、文件大小变小、I/O操作随机性等增加而加剧。另外，元数据服务器规模较大时，高并发性元数据访问会导致同步性能开销更加显著。目前，一些分布式文件系统采用高性能网络(如InfiniBand, GibE等)、SSD固态硬盘或SAN磁盘阵列、分布式共享内存(SMP或ccNUMA)等技术进行集群内部的元数据同步和通信。这的确可以明显提高系统性能以抵消同步开销，不过成本方面也徒然增加许多。

数据一致性，这是分布式系统必须面对的难题。分布式元数据服务模型同样面临潜在的系统发生错误的风险，虽然一部分元数据节点发生故障不会致使整个系统宕机，但却可能影响整个系统正常运行或出现访问错误。为了保证高可用性，元数据会被复制到多个节点位置，维护多个副本之间的同步具有很高的风险。如果元数据没有及时同步或者遭受意外破坏，同一个文件的元数据就会出现不一致，从而导致访问文件数据的不一致，直接影响到上层数据应用的正确性。这种风险发生的概率随着系统规模的扩大而大幅增加，因此分布式元数据的同步和并发访问是个巨大的挑战。使用同步方法对元数据进行同步，再结合事务或日志，自然可以解决数据一致性问题，然而这大大降低了系统的并发性，违背了分布式系统的设计初衷。在保证元数据一致性的前提下，尽可能地提高并发性，这就对同步机制和算法设计方面提出了严格要求，复杂性和挑战性不言而喻。

**无元数据服务模型**

既然集中式或分布式元数据服务模型都不能彻底地解决问题，那么直接去掉元数据服务器，是否就可以避免这些问题呢？理论上，无元数据服务模型是完全可行的，寻找到元数据查询定位的替代方法即可。理想情况下，这种模型消除了元数据的性能瓶颈、单点故障、数据一致性等一系列相关问题，系统扩展性显著提高，系统并发性和性能将实现线性扩展增长。目前，基于无元数据服务模型的分布式文件系统可谓凤毛麟角，Glusterfs是其中最为典型的代表。

对于分布式系统而言，元数据处理是决定系统扩展性、性能以及稳定性的关键。GlusterFS另辟蹊径，彻底摒弃了元数据服务，使用弹性哈希算法代替传统分布式文件系统中的集中或分布式元数据服务。这根本性解决了元数据这一难题，从而获得了接近线性的高扩展性，同时也提高了系统性能和可靠性。GlusterFS使用算法进行数据定位，集群中的任何服务器和客户端只需根据路径和文件名就可以对数据进行定位和读写访问。换句话说，GlusterFS不需要将元数据与数据进行分离，因为文件定位可独立并行化进行。GlusterFS独特地采用无元数据服务的设计，取而代之使用算法来定位文件，元数据和数据没有分离而是一起存储。集群中的所有存储系统服务器都可以智能地对文件数据分片进行定位，仅仅根据文件名和路径并运用算法即可，而不需要查询索引或者其他服务器。这使得数据访问完全并行化，从而实现真正的线性性能扩展。无元数据服务器极大提高了GlusterFS的性能、可靠性和稳定性。(Glusterfs更深入地分析请参考“[Glusterfs集群文件系统研究](http://blog.csdn.net/liuben/article/details/6284551)”一文)。

无元数据服务器设计的好处是没有单点故障和性能瓶颈问题，可提高系统扩展性、性能、可靠性和稳定性。对于海量小文件应用，这种设计能够有效解决元数据的难点问题。它的负面影响是，数据一致问题更加复杂，文件目录遍历操作效率低下，缺乏全局监控管理功能。同时也导致客户端承担了更多的职能，比如文件定位、名字空间缓存、逻辑卷视图维护等等，这些都增加了客户端的负载，占用相当的CPU和内存。

**三种元数据服务模型比较**

对Scale-Out存储系统而言，最大的挑战之一就是记录数据逻辑与物理位置的映像关系，即数据元数据。传统分布式存储系统使用集中式或布式元数据服务来维护元数据，集中式元数据服务会导致单点故障和性能瓶颈问题，而分布式元数据服务存在性能开销、元数据同步一致性和设计复杂性等问题。无元数据服务模型，消除了元数据访问问题，但同时增加了数据本身管理的复杂性，缺乏全局监控管理功能，并增加了客户端的负载。由此可见，这三种模型都不是完美的，分别有各自的优点和不足，没有绝对的优劣与好坏之分，实际选型要根据具体情况选择合适的模型，并想方设法完善其不足之处，从而提高分布式文件系统的扩展性、高性能、可用性等特性。集中式元数据服务模型的代表是Lustre, StorNext, GFS等，分布式元数据服务模型的典型案例有ISILON, GPFS, Ceph等，Glustrefs是无元数据服务模型的经典。以上这些都是非常强大的分布式文件系统，它们是非常好的设计典范。这也足以说明，架构固然非常关键，但具体实现技术却往往决定最后的结局。

**补充阅读**
[1] Ceph, [http://www.ibm.com/developerworks/cn/linux/l-ceph/index.html?ca=drs-](http://www.ibm.com/developerworks/cn/linux/l-ceph/index.html?ca=drs-)
[2] Glusterfs, [http://blog.csdn.net/liuben/article/details/6284551](http://blog.csdn.net/liuben/article/details/6284551)
[3] 集群NAS技术架构，[http://blog.csdn.net/liuben/article/details/6422700](http://blog.csdn.net/liuben/article/details/6422700)
> **转载：**[**刘爱贵的专栏**](http://blog.csdn.net/liuben/article/details/6749188)
