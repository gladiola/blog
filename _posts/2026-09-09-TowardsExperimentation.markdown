---
title:  "Towards Experimentation: SQL Injection and Quasi-Experimental Design"
date:   2026-09-09 08:30:00
description: Why SQL Injection defenses need to be tested like real experiments, and what William Shadish's quasi-experimental design work can teach network defenders.
---

When we get on the Internet, we're accessing programs that are the result of hundreds of hours of work put in by unseen programmers and content contributors worldwide. The machines we use to access the Internet are also the result of thousands of hours of labor by unseen factory workers, electrical engineers, petroleum refinery workers, plastics and metal manufacturing workers, and even rare-earth miners. We don't just stand on the shoulders of scientific giants of the past; the head that holds the daydreaming mind is cradled in the hands of our fellow person who has labored to bring us both vital data and idle joys. Many of us don't know how it all works.

Those who don't may live in fear of "The Hacker." Those of us who do understand programming recognize and respect the dangers, but know there are distinct technological limits to what people can and can't do with machines. Often, we recognize that the vulnerabilities exploited by attackers are really just overlooked or unaddressed, unlikely opportunities for problems with data. One man's disregarded trash of casual, unchecked input assignment is another man's treasured opportunity for exploit. In this post we deal with one of the most common problems associated with undisciplined programming techniques on the web: SQL Injection.

To understand SQL Injection's place in the world of web programming, we have to accept some basic descriptions of the programs that make web pages. There are three common kinds of content in the web programs involved: static, form-based, and dynamic content. Static content doesn't change between program runs. Form-based content responds to user interaction with an online form. Dynamic content changes as a result of program computations. We often see dynamic content that is based on user input from adjacent programs written to handle form-based content. [1]

In the mechanics of carrying data from one web program to another, attackers have found common points of manipulation. One such area is URL encoding. In the address bar of our browsers, we can see directory-based information that describes where files are stored, and also an opportunity to carry data from one program to another. It's in that carrying of data that attackers can seek to inject their own. Attacks that inject data for the purpose of striking at underlying databases fall into the category of SQL Injection attacks ("SQLIATK"). [2] [3]

SQL Injection attacks are some of the most common attacks on web programs. Functions that allow programmers to validate and sanitize inputs have been a common defense, available for a long time through widely circulated, standardized methods that are often official parts of published web languages like PHP. [4] Yet, despite the availability of these easy-to-use defense mechanisms, SQL Injection attacks continue to plague the web. An OWASP study of the Top 10 Web Vulnerabilities in 2004 listed "Injection Flaws" as 6/10. A similar listing by OWASP in 2013 placed "Injection" at number one. [3] As this was being written, OWASP was considering its Top 10 list for 2017. "Injection" was still at number one. [5] A combination of hasty programming, poor systems design, and lack of review and repairs seem to be the leading causes of promoting these vulnerabilities on the web.

By using SQL Injection, attackers can download, displace, or destroy data. The risk to users, customers, clients, businesses, and stakeholders is significant because these vulnerabilities are so easy to exploit that automated attack tools are publicly available and widely distributed. [7] SQL Injection attacks will remain a significant part of information security for the foreseeable future because, like many attacks on data in motion or data at rest, they are abuses of inherent characteristics. The same qualities that describe and define stateless transfer of data on the web create the very conditions to make a program susceptible to injection attacks. Because URL-encoded data transfers are a normal and legitimate part of web traffic, we will continue to see injection attacks occur.

Risk assessments are at the core of any effective information security policy. [8] Without correct threat modeling -- defining and describing expected methods of attack -- information technology professionals would not be able to mount an effective defense of their networks. Every day new attack tools surface. It's evident that skilled programmers can craft their own attacks using common programming tools and specialized libraries. [7] [3] [9] The most dangerous automated attack tools appear to be the most automated.

I feel that the most automated tools are the most dangerous because they imply a lower level of operator skill required to execute the programs. This supports a broad user base of attackers. Some of these programs come with online tutorials, manuals [10], and demonstrations [11]. Downloading the attack programs is often no more difficult than downloading any other program on the web. [12] At work I have witnessed the effects of these kinds of programs; the heavy use of automated attacks can lead to an availability problem for web programs. While the individual attacks themselves, hit-by-hit, may be ineffective, the deluge of rapid-fire requests as HTTP calls may still each require a response from the server, albeit dismissive.

Given the great variety of attack tools available and given the great variety of website structures available, how is a network defender able to effectively simulate plausible attacks in order to prepare a reasonable defense? This is a central question of some of my recent work, commercially and academically. In my work as a professional web programmer, I have seen a variety of effective and ineffective attacks against clients' sites. I suppose that responses to attacks and preventive attack mitigation techniques seem to be intuitive or instinctive. Are these responses supported by science? Do we see effective defense when the effects of previous attacks seem to stop? When responding to attacks of significant power, can we design and quickly carry out experiments that will prove our chosen defensive techniques are effective risk controls?

My focus here is the relationship between risk assessment and experimental design. My hope is to follow this research with a battery of examples illustrating these experimentation techniques, in order to demonstrate the correlation of these concepts. I am particularly interested in developing a methodology that would provide practical advisement or frameworks for decision making for small-shop programmers and sysadmins who need to rapidly develop, test, and deploy a defense mechanism.

Will it be possible to know, before experimentation begins in earnest, if the experiment can be designed to lead the programmer toward the right kind of answer? Can this outcome be promoted by choosing certain kinds of experiment types over one another? Can we show that hasty experimental design approaches can have immediate tactical benefit to the defender who must take immediate action? Can we show that a more deliberate experimental design can promote a stronger defense strategy by describing effective needed changes in security policies? How can programmers know if the sites they design in experimentation have a reasonable chance of producing the needed outcome to combat specific categories of threats? In order to reach toward effective answers, we have to begin by knowing that we are asking the right questions through experimental design.

# Related Work

We know that a standardized review of risk assessment categories can be found from NIST. Their Cybersecurity Framework lists core component functions that include "Identify, Protect, Detect, Respond, Recover." [13] When we consider the possibility of responding to a SQL Injection attack by mitigating with an experimentally proven technique, we can see participation in all of these functions. I find it helpful to consider them in reverse. To "Recover": "Improvements" would have been agreed upon already, as part of accepting the concept of responsive security maintenance and mitigation. To "Respond": "Mitigation" is directly listed. To "Detect": "Security Continuous Monitoring" can be chosen -- even though SQL Injection attacks attempt to strike at the database, they are most likely to be detected by filtering and examining logs of calls made to the website through URLs. To "Protect": "Information Protection Processes & Procedures" would be chosen, because periodic log reviews are a protection procedure. Finally, to "Identify": risk assessment -- the identification, observation, and response to risk from SQL Injection attacks -- would be chosen. This framework is useful to us as a paradigm for devising and implementing a mitigation response, because it is a widely acknowledged emerging government standard for communicating information security concerns throughout entities like governments and businesses. [13] [14]

Given the scope of our discussion here, I felt it was appropriate to concentrate on three of Shadish's chapters, roughly numbered 1, 4, and 11 in his major work on the subject.

The late William Shadish was an industry leader in describing quasi-natural experiments. [15] [16] A recent article by Benjamin Dean shows that Shadish's ideas can be readily applied to evaluating cybersecurity threats. [17] Shadish's work seemed to focus primarily on the social sciences and education [15] [18], but the constraints on experiments make his work applicable to establishing good experimental design for evaluating information security practices. There is an analogy available because many of the conditions in cyberattacks and social policy affect distinct units (like one person or one machine) with one-time events. Dean's work summarized methods that can be used to relate Shadish's experimental design guidance to larger situations involving policy changes. [17]

To accept the point Dean makes [17] that we should use quasi-natural experimentation because common, reproducible scientific experimentation methods may be too difficult or too complex for the contemporary network security environment, we would need to better understand Dean's underlying authority: Shadish. Shadish wasn't writing about computer programs; instead, he was writing about social programs and policies -- it was in that sense that Dean was able to apply Shadish's assertions to cybersecurity policies for nation-states. We, too, can adapt Shadish's advice on the relationship between quasi-natural experimentation and policy (i.e., "social program" [18]) in our quest for better, rapidly developed network defenses.

Shadish asserted in his article on the "Multiplist Perspective" that quasi-natural experimentation would benefit from critical multiplist perspectives: if multiple experiments are conducted with variations in subtask performance, we can gain the benefit of uncovering how bias might have affected our experiments. If we conducted every part of the experiment exactly the same way every time, our bias in choosing experimental frameworks would remain hidden. By accepting the prejudice of a too-rigid experimentation model, we might face the danger of rigorously hiding the detrimental effects of our own prejudices. [19]

In order to better understand the framework that Shadish's perspective can provide, I'd like to summarize some of his assertions from his book on Quasi-Experimental Designs. [20] Shadish's book contains several chapters important to fortifying the idea that quasi-natural experimentation is a good foundation for making scientific assertions.

Chapter 1 covered the general concept of quasi-natural experimentation in the broader scheme of scientific development: how experiments can describe cause, how experiments have evolved over time, contemporary experiment types, and a summary of different kinds of validity. This helps us understand that quasi-natural experimentation is a legitimate scientific method that fits inside the broader canon of past scientific achievement.

In Chapter 4, "Quasi-Experimental Designs That Either Lack a Control Group or Lack Pretest Observations on the Outcome," we can find guidance that might be helpful in situations where an attack is discovered by surprise. If we had a system whose state we knew or could prove before an attack, we could understand how subsequent experimentation might succeed or fail. So many attacks require situational defense without proper laboratory preparation.

In Chapter 11, "Generalized Causal Inference: A Grounded Theory," we have a chance at grasping Shadish's summary justification for using quasi-natural experimentation, discussing how principles of theory can support it.

Shadish attributed "the experiment" we accept today to the work of Fisher:

> "But when scientists started to use experiments in areas such as public health or education, in which extraneous influences are harder to control ... they found that the controls used in natural science in the laboratory worked poorly in these new applications. So they developed new methods of dealing with extraneous influence, such as random assignment (Fisher, 1925) or adding a nonrandomized control group (Coover & Angell, 1907). As theoretical and observational experience accumulated across these settings and topics, more sources of bias were identified and more methods were developed to cope with them (Dehue, 2000)." [20]

To Fisher and his contemporaries, we ascribe the concepts of random assignment and a nonrandomized control group -- as important to experimentation as laboratory glassware was to Chemistry. Shadish wrote:

> "Today, the key feature common to all experiments is still to deliberately vary something so as to discover what happens to something else later -- to discover the effects of the presumed causes." [20]

Shadish discussed the influence of Locke, Mackie, and Hume on the concepts of cause and effect, and the influence of Rubin and John Stuart Mill on causal relationships. Locke's definition:

> "A cause is that which makes any other thing, either simple idea, substance, or mode, begin to be; and an effect is that, which had its beginning from some other thing." [20]

Describing Mackie's work, Shadish told us about the "inus condition":

> "A lighted match is, therefore, what Mackie (1974) called an inus condition -- 'an insufficient but non-redundant part of an unnecessary but sufficient condition' ... It is part of a sufficient condition to start a fire in combination with the full constellation of factors." [20]

The inus condition is significant to us in exploring SQL Injection Attacks on complex networked systems, because practical experience shows us that so many smaller, contributing factors play a role but cannot be so strongly described as the cause themselves. I felt that this was particularly close to some practical conclusions I've seen drawn about cause and effect in computer security problems. We may say that a certain attack causes some specified damage, but the attack code is surrounded by many supporting inus conditions that are an assumed and often ignored part of the problem -- the file structure of the attacked program might be one example, the configuration details of a web server or database might be others. Shadish went on to warn:

> "Most causes are more accurately called inus conditions. ... This is one reason that the causal relationships we discuss in this book are not deterministic but only increase in probability that an effect will occur." [20]

He continued on effect, drawing on Hume:

> "An effect is the difference between what did happen and what would have happened." [20]

Counterfactual conditions ("A counterfactual is something that is contrary to fact" [20]) were noted in Shadish's discussion of Rubin's Causal Model. He concluded his discussion of cause and effect with John Stuart Mill's summary:

> "How do we know if cause and effect are related? In a classic analysis formalized by the 19th-century philosopher John Stuart Mill, a causal relationship exists if (1) the cause preceded the effect, (2) the cause was related to the effect, and (3) we can find no plausible alternative explanation for the effect other than the cause." [20]

Take Mill's third point, above, for example. How often do we presume that an attack was the cause of damage because we did not discover another cause? Assuming the correct identification of a cause by admitting that there are no other explanations can lead to dangerous omissions when human deceit is a factor. Since correct contamination identification is a strong predicate for implementing effective countermeasures, our incorrect assumptions can provoke an early quit to an investigation that might actually prolong or promote an attack by causing us to emplace the wrong corrections.

Shadish also discussed the old saw, "Correlation does not prove causation," and confounds. On manipulable and nonmanipulable causes, he wrote:

> "Experiments explore the effects of things that can be manipulated. ... Nonmanipulable events ... or attributes ... cannot be causes in experiments because we cannot deliberately vary them to see what happens. Consequently, most scientists and philosophers agree that it is much harder to discover the effects of nonmanipulable causes." [20]

I would suggest that this one-off nature of reality, when observing an attack against a website, goes to show that we witness nonmanipulable events. During the one event that is an attack, the specifics of the attack are normally beyond our control. However, Shadish does suggest support for simulating an attack to explore its cause, effect, and causal relationships, because "analogue experiments can sometimes be done on nonmanipulable causes, that is, experiments that manipulate an agent that is similar to the cause of interest." [20] In that, I would suggest, is justification for continuing with an experiment to simulate a SQL Injection attack, its detection, its defense, and its recovery.

Before he tells us about what kind of experiment might be at hand, Shadish covered causal description and causal explanation -- "the whole" or "the part" of cause as "molar" or "molecular" causal conditions -- offering important insight into instances when we might need to guard against projecting results into wider conclusions. [20] I assert that, depending on the behavior of the programmer at the time, the scientific interactions might take different forms of experiments: a "natural experiment" at the moment of attack, or a "correlational design, passive observational design, and nonexperimental design" [20] in various stages of logging, discovery, or normal business and machine operations related to the attack event. This might matter later, as quasi-experimental results are applied to patching activities and improving risk assessments: it will be critical to maintain correct context in order to understand why attacks succeed and why people and machines might fail or succeed in responding to them.

Shadish wrote:

> "In the end, then, causal descriptions and causal explanations are in delicate balance in experiments. What experiments do best is to improve causal descriptions; they do less well at explaining causal relationships." [20]

Causal description is about the consequences of varying treatment in an experiment.

> "The practical importance of causal explanation is brought home when [a treatment] ... fails to solve the problem. Explanatory knowledge then offers clues about how to fix the problem." [20]

Although we've already mentioned that the types of experiments we're interested in are "quasi-experiments," it's worth reviewing some of Shadish's established summary terms:

> "Experiment: a study in which an intervention is deliberately introduced to observe its effects." [20]
>
> "Randomized Experiment: An experiment in which units are assigned to receive the treatment or an alternative condition by a random process ..." [20]
>
> "Quasi-Experiment: An experiment in which units are not assigned to conditions randomly." [20]
>
> "Natural Experiment: Not really an experiment because the cause usually cannot be manipulated; a study that contrasts a naturally occurring event such as an earthquake with a comparison condition." [20]
>
> "Correlational Study: Usually synonymous with nonexperimental or observational study; a study that simply observes the size and direction of a relationship among variables." [20]

As Shadish discussed the features of a quasi-experiment, he noted:

> "Quasi-experiments share with all other experiments a similar purpose -- to test descriptive causal hypotheses about manipulatable causes ..." [20]
>
> "In quasi-experiments, the cause is manipulable and occurs before the effect is measured." [20]
>
> "In quasi-experiments, the researcher has to enumerate alternative explanations one by one, decide which are plausible, and then use logic, design, and measurement to assess whether each one is operating in a way that might explain any observed effect." [20]
>
> "Quasi-experimentation is falsificationist in that it requires experimenters to identify a causal claim and then to generate and examine plausible alternative explanations that might falsify the claim." [20]
>
> "Thus the focus on plausibility is a two-edged sword: it reduces the range of alternatives to be considered in quasi-experimental work, yet it also leaves the resulting causal inference vulnerable to the discovery that an implausible-seeming alternative may later emerge as a likely causal agent." [20]

Given those terms and ideas about quasi-natural experiments, I would assert that:

1. The moment of actual attack is a natural experiment.
2. The moment of discovery of an attack by observation or log review is a correlational study.
3. The systematic attempt to develop, test, or implement a defense against a SQLIATK in a testable, repeatable, observable manner is a quasi-experiment.

Accepting these concepts, let's continue by exploring why we might be interested in conducting experiments like these.

Returning to Dean's work, we can see that a "sprint" or round of experiments for the purpose of changing a policy might itself be regarded as a quasi-experiment. Dean discussed the impact of policy changes, particularly at the national level:

> "Quasi-natural experiments, by contrast, do not involve the random application of treatment. Instead, a treatment is applied due to social or political factors, such as a change in laws or implementation of a new government program. ... The group that receives treatment in a quasi-natural experiment is called the comparison group instead of the control group." [17]

Dean also advised readers to consider the cost of these kinds of experiments. While I would expect that, for academic purposes, we might initially see cost as a matter of number of test iterations or the monetary expense of machine setup and licensing, Dean related experimental design to cost directly: a more complex experimental design would be more expensive to run in order to affect a policy change ("... the robustness and generalizability of the results increases at the expense of practicality and/or cost" [17]). Dean advised that three feature types could broadly describe an experiment's complexity: (1) whether the study was prospective or retrospective; (2) whether the study used a control group, a comparison group, or neither; (3) comparison of data over time or over multiple characteristics. [17]

# Motivation

After seeing so many attacks against a commercial site I work with on a daily basis, I felt a natural interest in the attack tools available worldwide. In my research on effective and ineffective attacks against clients' commercial sites, I have found attack tools, author signatures in source code, ingenious page-serving mechanisms to imitate web server actions, instructional videos on SQL Injection attack tool use, websites publishing catalogs of known website defacements, and the like. There is a generous field of support for the casual attacker. As a defender, I see a marketplace crowded with expensive products produced by companies with months-long backlogs of threat-patching needs. I believe that the reality of our marketplace is that we have to be prepared to conduct defensive network security and application security experiments ourselves.

We know that large DDoS attacks like the Mirai botnet can reasonably make static defense inadequate. [21] We can see that, in general, cybersecurity threats have risen to the attention of the general public. Of these threats, different varieties of injection attacks, taken together, have a leading influence.

OWASP's website for its Top 10 of 2017 describes injection as:

> "Injection flaws, such as SQL, OS, and LDAP injection occur when untrusted data is sent to an interpreter as part of a command or query. The attacker's hostile data can trick the interpreter into executing unintended commands or accessing data without proper authorization." [5]

I learned about LDAP injection attacks by attending a security conference class given by Jared Haight. In a sample LDAP injection attack, we devised a PowerShell function that would check a username and password combination by contacting a computer with an LDAP call. If the username and password were valid, the spoofed LDAP call would work -- simple LDAP injection attacks could be used to verify user credentials, and that information could support other, escalated mischief within a network. [22] OWASP considers these injection variants just as harmful as SQLIATK, but due to our limits, we explore SQLIATK here.

Should we need to respond to the idea that injection attacks are worth securing against, my reading has included selections from Paul Day's *CyberAttack*. [23] Day pointed out contemporary factors showing that crime supported by computer abuse has made a substantial impact on many economies and cultures. We see the 2001 Budapest Convention as an international standard and mechanism for accepting the concept of computer crimes across nations and cultures. Day noted that the European Commission chose to adopt a "'general policy against cybercrime'" because there was "no agreed definition of cybercrime" in May 2007. He wrote:

> "The Commission's solution was to propose a threefold definition: 1. Traditional forms of crime such as fraud or forgery committed over electronic communication systems and information systems. 2. The publication of illegal content over electronic media (e.g. child sexual abuse material or incitement to racial hatred). 3. Crimes unique to electronic networks (e.g. attacks against information systems, denial of service and hacking)." [23] (p. 7)

Day wrote that, in the United States, "The FBI currently track cybercrime types using 27 different categories of cybercrime -- but there are more than 60 subcategories. Different law enforcement agencies across the world have different cybercrime laws -- and some countries have no laws at all." [23] (p. 7) Similarly, Dean noted:

> "... all countries might benefit on a net basis from international cooperation around increasing cybersecurity and fighting cybercrime. Yet such cooperation does not occur organically, even between countries or regions with commonly held values and interests, (e.g. the European Union and United States)." [17] (p. 146)

Day explained:

> "Cyber-criminals have quickly learned how to steal in cyberspace -- and the 'triangle of cybercrime' relies on three things: (1) the generation of possibilities for cybercrime through data theft and malware introduction; (2) the operation and maintenance of cyber-criminals through a secondary supply chain; and (3) a seemingly endless supply of low level cyber-criminals and 'cash-out mules' who are the foot soldiers in the cyber-mafia -- and who get caught eventually." [23] (p. 6)

In this scheme, our interest in evaluating SQL Injection Attacks relates to Day's first point: SQLIATK are a common mechanism of intrusion, and those injection-based intrusions are often the breach that leads to subsequent victimization from computer crimes.

Later in his book, after discussing the direct and indirect costs of cybercrime, Day summarized three studies commissioned by the US and UK governments between 2006 and 2012, offering several recommendations for preventing cybercrime. Among those relevant to our experiments:

> "1. Interfere with the distribution of crime-ware via filtering, automated patching and countermeasures against content injection attacks. Spam filters can prevent the delivery of deceptive messages" ... "Automated patching can make systems less vulnerable. Improved countermeasures to content injection attacks can prevent cross-site scripting and SQL injection attacks." [23] (p. 48)

This suggests support for the general idea of developing patches to systems subject to attack.

> "6. Interfere with the ability of the attacker to receive and use confidential data by encoding data in a form that renders it valueless to an attacker." [23] (p. 49)

This is significant to us because many standardized methods available in server-side programming languages already contain ways to neutralize an attacker's attempt to sniff out a target with simple matching, by encrypting the data before the attack occurs. Dictionary attacks, based on exhaustive iteration over word lists, can be foiled by breaking the ability of an attacker to discover a simple match within a system by encrypting the data beforehand. Similarly, navigation within a system can be foiled by jamming dictionary-style attacks that an attacker might use to navigate to commonly named directories, by using encrypted strings instead of plaintext names. We have witnessed these tactics in the wild, and they can be an attacker's obstacle in some of the experiments we might establish.

> "3. Prevent crime-ware from executing by validating code prior to execution." [23] (p. 49)

In large corporations we may see job-scheduling software used to initiate and validate a program run against a pre-planned schedule. For most small computer users, however, there aren't readily available code-validation procedures that would interfere with execution in a way that would stifle SQLIATK. Many attacks seem to be predicated on the idea that the attacker intends to abuse a legitimate system; the attack is often a momentary abuse of a common, working program. In these situations, the seemingly easy advice to validate code can actually turn into a complex problem. I would suggest that computers are good at following linear instructions, but presently poor at the kind of holistic, right-brain-style evaluation that people make in a moment. [24] In practice I have seen that identifying SQLIATK code can be done by directly reading server logs; as a person familiar with reading code, I can readily see the attack occurring in isolated lines. However, explaining to a computer, in principle, that each run of a server-side program was validated before it executes is a tall order. Instead, we often see web programming professionals opting for validating data input -- guided by client-side form-based suggestive limitations, server-side inspection and validation of received data, and maybe server-side inspection and validation of processed data prepared for output. This concept of code validation prior to execution is a rich area for exploration with experiments like these; preventing known attack code from executing might be an example of a quantifiable and qualifiable test that could yield simple binary ("yes/no") results lending itself well to simple mathematical scoring and statistical analysis.

With contemporary factors like these to consider, I think we can suggest that SQLIATK experiments are worthwhile. We can see that they can be societally beneficial by reducing the likelihood of successful computer crimes, even though we may not have a universal acceptance of what a computer crime is. We can see that there is probable cause for suggesting that advice for mechanisms to reduce computer crime, like Paul Day's suggestions above, could be testable as experiments by computer scientists.

# Experimental Structure

Let us consider the physical components to the computer environments that might support these experiments: the computer environment holding the operational programs, the mechanisms for conducting tests with and without human intervention, the structure of organized files holding data and programs throughout the experiment, and the choice of attack code.

For computer environments, I have encountered three main methods for simulating websites that looked reasonable for running attack simulations. The first, my favorite, was introduced to me at a B-Sides Charleston event on reversing and hacking Windows 7 applications with Kali Linux. [25] In these setups, a VirtualBox VM [26] is used to hold the target OS and another VM is used to hold the attacker's OS. [27] In another, VMWare virtual machines [28] were used to support the Samurai Web Testing Framework. For those attacks, Jason Gillam was able to demonstrate Burp Suite and other attack tools against a website especially built to be vulnerable -- a "hackable" site more like a target for CTF ("Capture The Flag") games used by hackers to compete for discovering target strings of code by successfully completing security puzzles. While robust and ready for hacking, these models are better suited for training security professionals than for running real-world experiments evaluating the effectiveness of SQLIATK tools; as a training tool for the people interacting in security, this "hackable site" model is clearly effective. [29] In the third model, I saw Jared Haight establish an effective class on security quickly by having a target site established in a Microsoft Azure environment [30], with subtle variations for each student's login. That method seemed to provide many people with sites quickly, and in a classroom environment I felt it was superior to having each student build his own set of VMs. But, whether hosted on another computer, "in the cloud," or on a local machine, using VMs was clearly an industry standard for establishing target environments. Using virtual machines as part of the experiment means there are some hardware limitations -- older equipment might not be suitable, though many CPUs manufactured after 2005 support virtualization. When considering these three main models for security experimentation that I have seen over the past year, I felt the best choice would be a model that would allow programmers to simulate their desired protected site, like a commercial client's website, or a section of it.

For testing mechanisms, we should choose programs that simulate but do not require human intervention. In contemporary web programming, graphical user interfaces are frequently implemented and may be related to attacks on web programming as either the attack program or the target; but their initial design -- the expectation that people would have to use the devices -- is not the only option. Many of those controls can be programmatically activated or simulated. To that end we can use programs like JUnit, QUnit, PHPUnit, or Selenium. [31] [32] [33] [34] This way, we could program the activation of controls and not rely as heavily on human test participants. We could also extract live attack code, modify it slightly, and send it using programs like `curl` with `cron`.

For the structure of the target and attacker file systems, we could consider four main components: (1) common Linux OS variants; (2) Microsoft Windows OS structures, like their Windows VM [35]; (3) a variety of database engines like MySQL [36] to support dynamic web pages and the main target for injections, along with supporting programs like phpMyAdmin [37]; (4) web server control panels, web server engines like Apache, and other related files.

To understand our expectations of the attack, we would need to decide how deeply we would simulate common patterns of SQLIATK. The targeted URLs may be a given, as there are target lists of vulnerable websites that circulate in forums on the Internet. The other stages would need to be determined: recon, dropper, RAT, sustained exploit, and exploited system integration. These phases of operation are common to attack attempts I have seen in the past, and would be influential components in any SQLIATK experimentation.

# The Road Ahead

I am looking forward to carrying out some of these experiments. I'm excited to get started with practical experimentation. Can we see the compromises that can occur during SQLIATK that I've witnessed in past attack and reversing classes? What will those attack successes look like? Will expected defensive conditions hold? Will expected weaknesses show failure? Very exciting.

Here, we have examined the support for the design of experiments with SQL Injection attack tools. We have shown that there is a scientifically valid pathway that can link experimental simulations to policy and practice improvements. We know that those policy changes can affect risk assessments by associating the type of event with appropriate risk categories. We have examined elementary terms and challenged our assumptions about them in order to make reasoned choices in our descriptions. We understand that despite the many complex influences upon systems we can draw valid scientific conclusions from experiments involving SQL Injection Attacks in a quasi-experiment. We recognize that the parameters of experimentation can change and be redefined by establishing a clearly defined scope of experiment. We understand that the complexity of experimental design is related to its costs. Given these ideas, I think we have established a reasonable foundation for describing and evaluating SQL Injection attack tools through quasi-natural experimentation in order to inform risk assessments and information security policies.

[//]: # (Hyperlinks)
[1]: ``
[2]: ``
[3]: ``
[4]: http://php.net/manual/en/security.database.sql-injection.php
[5]: https://owasp.org/index.php/Top_10_2013-Top_10
[7]: ``
[8]: ``
[9]: ``
[10]: https://www.scrt.ch/outils/mms/mms_manual.pdf
[11]: https://www.youtube.com/watch?v=uK3_qR3Kn_g
[12]: https://www.scrt.ch/en/attack/downloads/mini-mysqlat0r
[13]: ``
[14]: https://www.nist.gov/news-events/events/2017/03/cybersecurity-framework-virtual-events
[15]: http://faculty.ucmerced.edu/wshadish/shadish-cv-jan-2015
[16]: http://www.ipr.northwestern.edu/workshops/annual-summer-workshops/quasi-experimental-design-and-analysis/
[17]: ``
[18]: ``
[19]: ``
[20]: ``
[21]: ``
[22]: ``
[23]: ``
[24]: ``
[25]: https://www.kali.org/
[26]: https://www.virtualbox.org/
[27]: ``
[28]: http://www.vmware.com/
[29]: ``
[30]: https://azure.microsoft.com/en-us/?cdn=disable
[31]: http://junit.org/junit4/
[32]: http://qunitjs.com/
[33]: https://phpunit.de/
[34]: http://docs.seleniumhq.org/
[35]: https://www.microsoft.com/en-us/download/details.aspx?id=3702
[36]: https://www.mysql.com/
[37]: https://www.phpmyadmin.net/

**Works referenced above without a public URL:**

- [1] Ullman, Larry. *PHP for the Web.* Pearson Education, Peachpit Press, fourth edition, 2011. ISBN 978-0-321-73345-29000.
- [2] Tatroe, Kevin, Peter MacIntyre, and Rasmus Lerdorf. *Programming PHP.* O'Reilly, third edition, 2013. ISBN 978-1-449-39277-2.
- [3] Regalado, Daniel, et al. *Grey Hat Hacking: The Ethical Hacker's Handbook.* pp. 385-414. McGraw Hill Education, fourth edition, 2015. ISBN 978-0-183238-0.
- [7] Weidman, Georgia. *Penetration Testing: A Hands-On Introduction to Hacking.* No Starch Press, 2014. ISBN 978-1-59327-564-8.
- [8] Shelly, Gary B., and Harry J. Rosenblatt. *Systems Analysis and Design.* pp. 570-615. Course Technology, CENGAGE Learning, ninth edition, 2012. ISBN 978-1-133-27405-6.
- [9] Seitz, Justin. *Black Hat Python: Python Programming for Hackers and Pentesters.* No Starch Press, second printing, 2014. ISBN 978-1-59327-590-7.
- [13] "Framework for Improving Critical Infrastructure Cybersecurity." NIST, 2014.
- [17] Kampf, David, ed.; Dean, Benjamin. "Natural and Quasi-Natural Experiments to Evaluate Cybersecurity Policies," *Journal of International Affairs.* pp. 139-160. Winter 2016, Volume 70, Number 1. Columbia University.
- [18] Shadish, William R. Jr., Thomas Cook, and Laura C. Leviton. *Foundations of Program Evaluation: Theories and Practice.* pp. 36-67. Sage Publications, first printing, 1991. ISBN 0-08-039355-1.
- [19] Lipsey, Mark, ed.; Trochim, William, ed.; Shadish, William R., Thomas D. Cook, Arthur C. Houts. "Quasi-Experimentation in a Critical Multiplist Mode," *Advances in Quasi-Experimental Design and Analysis, New Directions for Program Evaluation.* pp. 29-46. American Evaluation Association, 1986.
- [20] Shadish, William, et al. *Experimental and Quasi-Experimental Designs for Generalized Causal Inference.* Wadsworth, CENGAGE Learning, 2002. ISBN 978-0-395-61556-0.
- [21] "Internet of Things DDoS White Paper." Electricity Information Sharing and Analysis Center.
- [22] Haight, Jared. "Introduction to Powershell and How to Use It for Evil." Information Security Lecture sponsored by B-Sides Charleston. Charleston, SC, USA. April 2017.
- [23] Day, Paul. *CyberAttack: The truth about digital crime, cyber warfare and government snooping.* Carlton Books, 2014. ISBN 978-1-78097-533-7.
- [24] Edwards, Betty. *Drawing on the Right Side of the Brain: A Course in Enhancing Creativity and Artistic Confidence.* Jeremy P. Tarcher, Inc., St. Martin's Press, 1989. ISBN 978-0-87477-513-2.
- [27] Rodgers, Doug. Twitter: @pandatrax. "Beginner Exploit Writing," B-Sides Charleston class, October 2016.
- [29] Gillam, Jason. "Web Penetration and Testing," B-Sides Charleston, 11 NOV 2016.
