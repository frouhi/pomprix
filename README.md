# pomprix

<!--
  Title: Pricing products powered by LLMs
  Description: A low friction pricing model that keeps your costs low and predictable
  Author: frouhi
  Tags: LLM, GPT, GPT-4, chatbot, chatgpt, openai
  -->

A low friction pricing model that keeps your costs low and predictable

Pricing software powred by large language models is hard. The token based cost regime that governs models such as openAI GPT-4, can lead to a few power users dominating our total costs. Traditional SaaS pricing is insufficient to safeguard against high and unpredictable costs. On the other hand, charging based on metered usage can make interpreting costs difficult for customers. In this article we will propose a combination of subscription and smart throttling as a solution. We will also introduce our LangChain inspired SDK as an easy way to incorporate this method into your code base.

![2](https://github.com/frouhi/pomprix/assets/34617952/80fcc9f2-b30c-4294-909e-b9090d1279af)

## Challenges
There are several factors that can contribute to high costs associated with LLMs in an application:
Using larger and more resource intensive models such as GPT-4
Large prompts
heavy usage patterns such as chatbots
There are some optimizations that can reduce the costs, such as optimizing prompts, but the costs can still remain high. More importantly, the costs can be highly unpredictable. If we break down the cost by users, we can see that a few power users can have a large impact on the total cost. In this section we will explore two existing pricing models, and their insufficiencies.

### 1. Subscription Pricing Model

This pricing model has been popular due to the reliable recurring revenue it provides. This model is easy to implement, and is easy to underestand by customers, leading to a frictionless experience. However, this model in the context of LLMs is not robust against outliers. Imagine a few power users spending 10 hours each day watching netflix. This increases the load marginally and doesn’t have a material impact on the costs. However, if a few power users spend 10 hours each day conversing with a chatbot powered by GPT-4, the costs can increase in a meaningful way.
### 2. Metered Pricing Model

In this model the customer receives a bill at the end of period based on their usage amount. This method reduces the costs and makes them more robust to outliers. However, implementing this metering system can be resource intensive. Additionally, communicating how the customer is being charged can be challenging, leading to friction in the checkout process. Understanding token based pricing can be non-trivial even for programmers.
Note that we can also combine the two models by allowing some amount of usage with a monthly subscription, while charging for surplus usage. This method however, still have the challenges of explaining how the surplus is charged, and the complexities of implement the metering mechanism.

## Our Solution

We propose subscription in conjunction with a smart throttling mechanism.

![1](https://github.com/frouhi/pomprix/assets/34617952/cc607444-b377-40db-b100-aca9533b40e8)

The subscription lower the checkout friction for customers. The smart throttling mechanism has the goal of not surpassing spending limits set by the business, with the minimum impact to the lowest number of customers. This method only targets a few power users, and even for them throttling happens only at some peak times as necessary. More importantly, the throttling is applied slowly and incrementally. At lower levels, it is not noticeable and is usually enough to lower the spike. It will be increased only if necessary as needed. The customer has the option of paying a fee and removing the throttling for 24 hours.
Implementation using our SDK: PomPrix
We are in the process of developing and finalizing PomPrix, a Python and Javascript SDK that provides a simple way to implement this solution. The SDK is very similar to LangChain and implements the same interface for the most part. The easy to deploy backend provides the databases and background analytics processes needed for this system. Here is a simple pre-release preview:
```
import asyncio
import time
import sys
from pomprix import PPRunnable


async def stream(runnable):
    async for i in runnable.astream("count from 1 to 50", user_id=1, info=True):
        if type(i) is dict:
            print(" >>", i, end="", flush=True)
            continue
        print(i, end="", flush=True)


def run_test(iter=10):
    pomprix = PPRunnable("http://localhost:8000/pomprix/")
    for _ in range(iter):
        start = time.time()
        asyncio.run(stream(pomprix))
        print(f" [{time.time() - start}]\n")


if __name__ == "__main__":
    if len(sys.argv) > 1:
        run_test(int(sys.argv[1]))
    else:
        run_test()
```
The output is as follows:

```
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {} [1.2807340621948242]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {} [1.0529298782348633]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {'confidence_level': 99.9, 'stripe_link': 'https://buy.stripe.com/test_28odU91ohfLubNmfqc'} [3.268908977508545]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {'confidence_level': 99.9, 'stripe_link': 'https://buy.stripe.com/test_28odU91ohfLubNmfqc'} [2.2311859130859375]
```
the confidence level is the output of a conformal prediction model telling us how confident the system is in that if the usage is reduced at this level, the system will remain bellow the spending limit. A lower confidence level means a higher throttling duration. This can be used to let the user know their usage is being throttled at lower levels by, for example, turning the border around the chat window yellow.
Additionally, the info also contains a stripe link. The moment the purchase is completed through this link, the throttling is removed for 24 hours.
```
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {'confidence_level': 99.9, 'stripe_link': 'https://buy.stripe.com/test_28odU91ohfLubNmfqc'} [2.254591226577759]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {'confidence_level': 99.9, 'stripe_link': 'https://buy.stripe.com/test_28odU91ohfLubNmfqc'} [2.2376418113708496]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {} [1.2916250228881836]

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50 >> {} [1.252641201019287]
````
Note that PomPrix is not officially released to public yet. For questions, comments, or if you are interested in this solution, you can reach us through this [form](https://docs.google.com/forms/d/e/1FAIpQLSdw6axmbZ_-KkRMzW5uQL7gPkVbu3M4f_lYhXU_6Hva9bkamw/viewform) or by contacting @frouhi. The client code will be available here soon, but please let us know if you are interested before release.

## Conclusion

Pricing products powered by LLMs is hard. Traditional subscription or metered pricing models are not sufficient for token based pricing of underlying LLMs. In this article, we explored subscription in conjunction with a smart throttling system as a solution.
