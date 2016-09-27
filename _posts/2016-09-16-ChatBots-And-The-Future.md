---
layout:     post
title:      ~
date:       2016-07-24  15:31:19
summary:    My Views On Chat-bots
categories: Web Interaction
---
<div style="text-align:center" markdown="1">

![desk](https://upload.wikimedia.org/wikipedia/en/3/39/R2-D2_Droid.png)

</div>
Greetings earthling,




I wanted to talk about chat bots today. I find them to be a really interesting concept that have the capability to revolutionize the web and app space. It represents a fundamental change in how we interact with web services.

Chat bots, also called Conversational Agents or Dialog Systems, are a hot topic. Microsoft is making big bets on chat bots, and so are companies like Facebook (M), Apple (Siri), Google, WeChat, and Slack. There is a new wave of startups trying to change how consumers interact with services by building consumer apps like Operator or x.ai, bot platforms like Chatfuel and bot libraries like Howdy’s Botkit. Microsoft recently released their own bot developer framework.

This article will be divided into 2 parts. The first one will talk about what chat bots represent ,the good and the bad. The second half will be devoted to understanding the technology behind these bots.

# The Good And The Bad

## Wait, but can the bot .. ?

* ..handle race and cultural issues? The world is extremely diverse and there are differing opinions on various topics. What might sound right to one person might not necessarily apply to another. This brings in other problems like handling the way different people of differing cultures talk,  if interactivity is such an important issue, this issue needs to be handled to make a chat bot truly successful.

* The illusion of another person talking to you could lull you to a false sense of security. This could lead you to share private/personal information with the bot. How should the chat bot handle this information? This is also something that needs to be given a fair amount of thought. Edward Snowden talked about this very topic in detail [**here**](http://fortune.com/2016/09/22/google-allo-nope/) about Google's smart bot Allo. It's a very interesting read.

* I think the most important issue with chat bots is that the technology isn't simply there yet. We have a long way to go. To make a chat bot truly interactive and able to hold a conversation with an adult, there is a long way to go. Humans as such don't have a guided state machine in their conversation. Humans can go wildly off course. The bot should be able to handle those with no issues. Like a bad joke ruins the conversation, the bot making a obvious mistake can destroy the illusion (the magic is gone)

* The chat bot works on the principle of continuous feedback. If that feedback is corrupted say by some users, the whole model goes into a direction that is not wanted. For instance, when Microsoft released a chat bot , the bot began to reply with racist and sexist comments after a few hours of user interaction. These conversations led to the bot learning something that it wasn't meant to. So a bot should be able to decipher what should and shouldn't be learned, which in itself is a very difficult problem to solve.

* Is there a perfect human? If a chat bot needs to mimic a human, definitions of a perfect human differ from one person to another. Can a chat bot capture all those requests and needs? This is very thought provoking and really brings into scope of how diverse a species we are.

Overall, there are a lot of factors that need to be perfectly balanced to make the bot work and will take a great deal of effort and breakthroughs. Though that shouldn't be a reason to abstain us from delve into developing the technology.Which leads us to the next section


## The Future Looks Great !

* Chat bots have the capability to revolutionize web interaction, imagine a world where a travel and booking company only relies on a chat bot to handle all travel operations. Planning your trip for you all inside that chat window which would include booking, hotel/destination selection and payment. That is indeed a very powerful idea.

* A chat bot if trained in conversation with esteemed personalities could act like your personal psychiatrist or personal secretary. Recently twitch users have used chat bots to reply to friends (to act as a chat moderator).This represents a very innovative use case, famous personalities, organizations can use a chat bot trained on their conversations to reply to their millions of followers. The possibilities are endless from customer support to entertainment.

* Chat bots could also derive opinions on a topic and capture the sentiment while conversing with them .This information could in turn lead to useful applications like perceiving public sentiment leading to more awareness as to what is going on in the world in the case of how people are feeling.

## The Current Picture and Motivations

The primary motivation of chat bots is that 90% of our time on mobile is spent on email and messaging platforms. Following the convention of going to where the users are, services offered through a messaging platform are more likely to connect. But is this really true? Do humans want to have a conversation with a machine? Or do they simply like to click through buttons to achieve their objective. The answer is , it depends . A lot of tasks don't need a chat bot to handle them, the space for button and click related tasks definitely have a dominant place on the web. But there is indeed a very specific space that can be exploited very effectively by these bots.

Nowadays bots are used with state guided mechanisms to conversation to keep the conversation on track. Examples that come into mind are as of today are Google Allo . It can answer questions regarding general knowledge, movies, weather, set reminders, play games, add subscriptions, book tickets, give directions and a lot more. Though it uses suggestions to guide the conversation in the direction Google wants it to head, it is meant to represent the pinnacle of what Chat bots can achieve at this stage.
Suggestions from chat bots are a necessary evil required at this stage.


# Technology

Let's talk about the technology that powers these chat bots.
Artificial Intelligence/Machine Learning and more specifically deep learning are at heading this quest. The field has seen a lot of advancements in the past few years and is blowing up right now. It would be safe to say that we would be in the 'AI bubble' presently.
Neural networks form the basis of these deep learning algorithms More specifically Recurrent Neural Networks (RNN's). RNN's are really hot right now and are used in almost all NLP based tasks by the organizations that are heading the chat bot effort.

## Recurrent Neural Netorks
Humans don’t start their thinking from scratch every second. As you read this article, you understand each word based on your understanding of previous words. You don’t throw everything away and start thinking from scratch again. Your thoughts have persistence.

Traditional neural networks can’t do this, and it seems like a major shortcoming. For example, imagine you want to classify what kind of event is happening at every point in a movie. It’s unclear how a traditional neural network could use its reasoning about previous events in the film to inform later ones.

Recurrent neural networks address this issue. They are networks with loops in them, allowing information to persist.

###image
![desk](http://d3kbpzbmcynnmx.cloudfront.net/wp-content/uploads/2015/09/rnn.jpg)

RNNs have difficulties learning long-term dependencies – interactions between words that are several steps apart. That’s problematic because the meaning of an English sentence is often determined by words that aren’t very close: “The man who wore a wig on his head went inside”. The sentence is really about a man going inside, not about the wig. But it’s unlikely that a plain RNN would be able capture such information.
To solve this issue LSTM's were developed.

## LSTM
Long Short Term Memory networks – usually just called “LSTMs” – are a special kind of RNN, capable of learning long-term dependencies. They were introduced by Hochreiter & Schmidhuber (1997), and were refined and popularized by many people in following work.1 They work tremendously well on a large variety of problems, and are now widely used.

###image
![desk](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/Long_Short_Term_Memory.png/600px-Long_Short_Term_Memory.png)

LSTMs are explicitly designed to avoid the long-term dependency problem. Remembering information for long periods of time is practically their default behavior, not something they struggle to learn!

Chat bots use a variation of RNN's by using Sequence To Sequence Models. A basic sequence-to-sequence model, as introduced in Cho et al., 2014 (pdf), consists of two recurrent neural networks (RNNs): an encoder that processes the input and a decoder that generates the output. This basic architecture is depicted below.

![desk](http://d3kbpzbmcynnmx.cloudfront.net/wp-content/uploads/2016/04/nct-seq2seq.png)

This [article](http://www.wildml.com/2016/01/attention-and-memory-in-deep-learning-and-nlp/) is a great read.

# Types Of Chat Bots

##RETRIEVAL-BASED VS. GENERATIVE MODELS

Retrieval-based models (easier) use a repository of predefined responses and some kind of heuristic to pick an appropriate response based on the input and context. The heuristic could be as simple as a rule-based expression match, or as complex as an ensemble of Machine Learning classifiers. These systems don’t generate any new text, they just pick a response from a fixed set.

Generative models (harder) don’t rely on pre-defined responses. They generate new responses from scratch. Generative models are typically based on Machine Translation techniques, but instead of translating from one language to another, we “translate” from an input to an output (response).


Deep Learning techniques can be used for both retrieval-based or generative models, but research seems to be moving into the generative direction. Deep Learning architectures like Sequence to Sequence are uniquely suited for generating text and researchers are hoping to make rapid progress in this area. However, we’re still at the early stages of building generative models that work reasonably well.

You can read more about deep learning and chat bots [here](http://www.wildml.com/2016/04/deep-learning-for-chatbots-part-1-introduction/)


##LONG VS. SHORT CONVERSATIONS

The longer the conversation the more difficult to automate it. On one side of the spectrum are Short-Text Conversations (easier) where the goal is to create a single response to a single input. For example, you may receive a specific question from a user and reply with an appropriate answer. Then there are long conversations (harder) where you go through multiple turns and need to keep track of what has been said. Customer support conversations are typically long conversational threads with multiple questions.

##OPEN DOMAIN VS. CLOSED DOMAIN

In an open domain (harder) setting the user can take the conversation anywhere. There isn’t necessarily have a well-defined goal or intention. Conversations on social media sites like Twitter and Reddit are typically open domain – they can go into all kinds of directions. The infinite number of topics and the fact that a certain amount of world knowledge is required to create reasonable responses makes this a hard problem.

In a closed domain (easier) setting the space of possible inputs and outputs is somewhat limited because the system is trying to achieve a very specific goal. Technical Customer Support or Shopping Assistants are examples of closed domain problems. These systems don’t need to be able to talk about politics, they just need to fulfill their specific task as efficiently as possible. Sure, users can still take the conversation anywhere they want, but the system isn’t required to handle all these cases – and the users don’t expect it to.



#Conclusion

![desk](http://www.pocketpc.ch/attachments/samsung-galaxy-note-10-1-2014-edition/144879d1386977823-rom-sm-p605-hal-9000-slim-p605xxubmj9-tumblr_mxn9oy4c1r1rulnxgo1_500.jpg)

To  end this long and meandering blog post, I would just like to say that the future is indeed bright for bots and intelligence in general.
The long term goal for computers must be to solve Intelligence. Solve Intelligence. Use that to make the world a better place. This Deep Mind motto really inspires me. These advancements really make me believe that we are on the right path and recent chat bots represent a milestone in that journey.




Cheers
