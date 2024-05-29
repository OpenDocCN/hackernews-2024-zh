<!--yml
category: 未分类
date: 2024-05-27 15:11:23
-->

# Is Cloud the new mainframe?. IBM mainframes were the original onsite… | by Billy Newport | Medium

> 来源：[https://medium.com/@billynewport/is-cloud-the-new-mainframe-e43133cac151](https://medium.com/@billynewport/is-cloud-the-new-mainframe-e43133cac151)

# Is Cloud the new mainframe?

IBM mainframes were the original onsite private cloud. Racks of integrated compute and storage running a bespoke software stack that are installed onsite. Customers liked them because they offered an integrated solution which offered pay as you go elastic scaling. They were overprovisioned and a call to IBM would unlock more capacity. IBM maintained the hardware and extensive hot swap capabilities and sparing meant very high uptime and easy hardware swaps when failures happened. IBM charged for service usage and storage usage. Pay as you go. Start cheap and your costs increase as your usage increases.

If you reached the capacity limits of installed capacity it was reasonably cheap and easy to upgrade the onsite hardware to have more capacity. Multiple data centers, replication are all supported.

Crazy high availability numbers which today's cloud vendors can only dream of. Advanced hardware support for many common failures including cpu failure with hot swapping…

The software stack running on them was well integrated. Applications, storage, databases and so on all running using a manager to keep track of relative priorities and budgets.

IBM has, still and will continue to make a lot of money from its mainframe customers. Anybody renting mainframe capacity to run applications basically keeps paying every month. Once a customer builds an application or application suite on mainframe then it’s difficult or expensive to move to another platform so it’s sticky and the monthly payments keep coming in to IBM. If a new function is needed, the easiest place to do it is likely on the mainframe with everything else.

It was a brilliant business model. Building applications and ecosystems to run on this platform lowered develop costs and gave businesses elastic pricing on the platform as their business grew.

The problem with it was any heavily used applications paid service prices proportional to their monthly usage of services. As the application usage grows, the bill grows and the control of the bill is largely in IBM’s hands. You use more, you pay more. If a company runs 100 applications on the mainframe then it’s inevitable the bulk of the mainframe monthly cost will be a handful of those applications. Applications almost always start small. The focus is on function and not on optimization. So, it ships fast and doesn’t cost a lot as it’s lightly used. But, a small handful of applications become popular and are easy to scale. Unfortunately, while compute is elastic, budgets are not…

So, heavily used applications became expensive to operate. Even if you stop developing it, you are still paying dollars out the door to run it on the mainframe. Typically, developers are assigned to optimize expensive applications as they grew to try and optimize them to lower costs. There are 2 phases for most successful applications, build and stabilize it and then phase 2 is manage costs as it grows, you want the costs to increase much slower than your usage. Inevitably, customers try to migrate workloads from the mainframe to “cheaper” platforms but these projects can be very expensive to do and they do fail more often than people realize. Reproducing the non functional features provided by the mainframe is also more expensive than most customer realize, it’s never a simple recompile and run scenario. Even reproducing an application using the current cool languages and tools is very difficult to do.

What did work well I think on minicomputers (AS/400 or iSeries) and mainframes were places to hosted ISV applications. We call hosted ISV applications SaaS platforms today :) I think these were usually cost effective. The customer got a great functional stack for some valuable business purpose running on best of class infrastructure.

# Today's Cloud kind of looks exactly the same as the mainframe scenario.

If you stand back and look at it. The prior section also describes today's cloud and predicts the same issues. Companies have rushed to get on the cloud with the cool kids. I predict many companies will try to rush to reduce cloud expenditure and will find migrating onsite to be an expensive proposition if it’s even possible.

I see cloud being mostly valuable for running SaaS, not for close to metal services. Think CRM/Payroll/development hosting/MS Office/Corporate email and so on, not data storage/kubernetes/virtual machine hosting. This is a higher value to a company than just hosting data or virtual machines.

If your company still hasn’t done a big cloud move then I think there is a high probability you will end up looking very smart rather than slow in the short/medium term…