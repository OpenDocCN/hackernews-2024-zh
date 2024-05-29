<!--yml
category: 未分类
date: 2024-05-27 14:26:10
-->

# Distributed Inference and Fine-tuning of Large Language Models Over The Internet

> 来源：[https://browse.arxiv.org/html/2312.08361v1](https://browse.arxiv.org/html/2312.08361v1)

0:  prefix_tokens, embeddings, known_servers

1:  generated_sequence = list()

2:  cache = dictionary()

3:  streams = dictionary()

4:  chain = find_best_chain(known_servers)

5:  for <math alttext="\text{server}\in\text{chain}" class="ltx_Math" display="inline" id="alg1.l5.m1.1"><semantics id="alg1.l5.m1.1a"><mrow id="alg1.l5.m1.1.1" xref="alg1.l5.m1.1.1.cmml"><mtext id="alg1.l5.m1.1.1.2" xref="alg1.l5.m1.1.1.2a.cmml">server</mtext><mo id="alg1.l5.m1.1.1.1" xref="alg1.l5.m1.1.1.1.cmml">∈</mo><mtext id="alg1.l5.m1.1.1.3" xref="alg1.l5.m1.1.1.3a.cmml">chain</mtext></mrow><annotation-xml encoding="MathML-Content" id="alg1.l5.m1.1b"><apply id="alg1.l5.m1.1.1.cmml" xref="alg1.l5.m1.1.1"><ci id="alg1.l5.m1.1.1.2a.cmml" xref="alg1.l5.m1.1.1.2"><mtext id="alg1.l5.m1.1.1.2.cmml" xref="alg1.l5.m1.1.1.2">server</mtext></ci><ci id="alg1.l5.m1.1.1.3a.cmml" xref="alg1.l5.m1.1.1.3"><mtext id="alg1.l5.m1.1.1.3.cmml" xref="alg1.l5.m1.1.1.3">chain</mtext></ci></apply></annotation-xml><annotation encoding="application/x-tex" id="alg1.l5.m1.1c">\text{server}\in\text{chain}</annotation><annotation encoding="application/x-llamapun" id="alg1.l5.m1.1d">server ∈ chain</annotation></semantics></math> do

6:     streams[server] = rpc_inference(server)

7:     cache[server] = list()

8:  end for

9:  

10:  inputs = embeddings(prefix_tokens)

11:  while should_continue(generated_sequence) do

12:     tail_servers = copy(chain)

13:     while not empty(tail_servers) do

14:        server = tail_servers.pop_left()

15:        try:

16:              <math alttext="\triangleright" class="ltx_Math" display="inline" id="alg1.l16.m1.1"><semantics id="alg1.l16.m1.1a"><mo id="alg1.l16.m1.1.1" xref="alg1.l16.m1.1.1.cmml">▷</mo><annotation-xml encoding="MathML-Content" id="alg1.l16.m1.1b"><ci id="alg1.l16.m1.1.1.cmml" xref="alg1.l16.m1.1.1">▷</ci></annotation-xml><annotation encoding="application/x-tex" id="alg1.l16.m1.1c">\triangleright</annotation><annotation encoding="application/x-llamapun" id="alg1.l16.m1.1d">▷</annotation></semantics></math>

Attempt normal inference

17:              outputs = streams[server].send(inputs)

18:              cache[server].append(inputs)

19:              inputs = outputs

20:        catch ServerFailed:

21:              <math alttext="\triangleright" class="ltx_Math" display="inline" id="alg1.l21.m1.1"><semantics id="alg1.l21.m1.1a"><mo id="alg1.l21.m1.1.1" xref="alg1.l21.m1.1.1.cmml">▷</mo><annotation-xml encoding="MathML-Content" id="alg1.l21.m1.1b"><ci id="alg1.l21.m1.1.1.cmml" xref="alg1.l21.m1.1.1">▷</ci></annotation-xml><annotation encoding="application/x-tex" id="alg1.l21.m1.1c">\triangleright</annotation><annotation encoding="application/x-llamapun" id="alg1.l21.m1.1d">▷</annotation></semantics></math>

Replace the failed server

22:              streams.pop(server).close()

23:              past_inputs = cache.pop(server)

24:              new_servers = replace_failed_server(

25:                    server, past_inputs, cache,

26:                    streams, known_servers)

27:              chain.replace(server, new_servers)

28:              tail_servers.push_left(new_servers)

29:     end while

30:     

31:     logits = compute_logits(outputs, embeddings)

32:     next_token = choose_next(logits) {e.g. greedy}

33:     generated_sequence.append(next_token)

34:     inputs = embeddings(next_token)

35:  end while

36:  

37:  for <math alttext="\text{server}\in\text{chain}" class="ltx_Math" display="inline" id="alg1.l37.m1.1"><semantics id="alg1.l37.m1.1a"><mrow id="alg1.l37.m1.1.1" xref="alg1.l37.m1.1.1.cmml"><mtext id="alg1.l37.m1.1.1.2" xref="alg1.l37.m1.1.1.2a.cmml">server</mtext><mo id="alg1.l37.m1.1.1.1" xref="alg1.l37.m1.1.1.1.cmml">∈</mo><mtext id="alg1.l37.m1.1.1.3" xref="alg1.l37.m1.1.1.3a.cmml">chain</mtext></mrow><annotation-xml encoding="MathML-Content" id="alg1.l37.m1.1b"><apply id="alg1.l37.m1.1.1.cmml" xref="alg1.l37.m1.1.1"><ci id="alg1.l37.m1.1.1.2a.cmml" xref="alg1.l37.m1.1.1.2"><mtext id="alg1.l37.m1.1.1.2.cmml" xref="alg1.l37.m1.1.1.2">server</mtext></ci><ci id="alg1.l37.m1.1.1.3a.cmml" xref="alg1.l37.m1.1.1.3"><mtext id="alg1.l37.m1.1.1.3.cmml" xref="alg1.l37.m1.1.1.3">chain</mtext></ci></apply></annotation-xml><annotation encoding="application/x-tex" id="alg1.l37.m1.1c">\text{server}\in\text{chain}</annotation><annotation encoding="application/x-llamapun" id="alg1.l37.m1.1d">server ∈ chain</annotation></semantics></math> do

38:     streams[server].close()

39:  end for

40:  return generated_sequence