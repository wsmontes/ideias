# What Will Kill the Liberation Client

## (Steve Jobs, after reviewing the project)

---

**[Stage. Black turtleneck. Jeans. He picks up a printed spec, flips through it silently for ten seconds. Drops it on the table.]**

I read it. All of it. The philosophy. The principles. The 30-page threat model. The wireframes. The governance document. The community curation roles.

And I have to tell you something you don't want to hear.

This product is going to die. Not because the idea is bad -- the idea is beautiful. But because you're so in love with the idea that you've forgotten what it takes to actually make something someone *wants to use*.

Let me tell you the three things that will kill this app.

---

**ONE: THE SIX-PROTOCOL SUICIDE PACT.**

You're launching with ActivityPub. AT Protocol. Nostr. Matrix. IRC. XMPP. Six protocols. And I'll tell you exactly what that means: six times the bugs. Six times the edge cases. Six protocols where something will break, and six reasons for a user to say "this doesn't work" and delete the app forever.

You think you're building breadth. You're building fragmentation. Every protocol you add doesn't make the product better -- it makes it heavier. It dilutes the experience. It turns a beautiful idea into a science project.

Pick one. Make it perfect. Then add the second.

ActivityPub has 10 million users. Mastodon is the flagship of the fediverse. Start there. Make the best Mastodon client anyone has ever seen. Get every detail right. Make it so good that people switch from their current client *just to have it*.

Then add Nostr. Then Bluesky. Not before.

If you launch with six, you'll launch with zero -- because none of them will work well enough. And the seventh thing that won't work? The offline queue. The sync engine. The identity linking. Because you're spending your engineering budget on protocol adapters instead of making the core experience flawless.

**The hard decision is not what to build. The hard decision is what to say no to. You're saying yes to everything. That's not vision. That's cowardice.**

---

**TWO: THE EMPTY ROOM.**

I open the app. I've just installed it. I've gone through your six-part onboarding flow. I've connected my Mastodon account. I've generated my Nostr keypair. I've confirmed my Matrix homeserver.

Now what?

**[He pauses. Looks at the imaginary phone in his hand.]**

Nothing. There's nothing there. I'm in a room with no people. No posts. No conversation. I see my beautiful unified timeline with protocol badges and encryption indicators -- and it's empty.

Nobody wants to be first at a party. You open the app, there's nothing there, you close it, and you never open it again.

Your Tasting Menu is clever. 20 curated posts cross-protocol. That buys you three minutes. Your Starter Packs are clever. That buys you five more minutes. But they're band-aids on a bullet wound.

The real question is: what does the user see in the first THREE SECONDS?

If I don't see something that makes me feel like I've arrived somewhere worth being, you've lost me. Not in a week. Not in a day. In three seconds.

You know who solved this? Twitter in 2007. You signed up, they showed you 50 suggested accounts. You followed ten. Suddenly your timeline was alive. Did they have fancy starter packs? No. They had a curated list of interesting people. One list. That worked.

You're building a protocol bridge but you haven't solved the first problem: **getting people into the room.**

The offline-first architecture makes this harder, not easier. Because posts don't live on a server I can query. They live on the device. So when I join, I'm not just alone -- I'm alone in a way that's fundamentally architectural.

You need to answer this before you write one more line of sync engine code. What happens in the first three seconds?

---

**THREE: YOU HAVE NO OPINION.**

Great products have strong opinions. They say no to almost everything. The iPhone doesn't have a keyboard. It doesn't have a stylus. It doesn't have Flash. It doesn't have a replaceable battery. Every one of those "no" decisions was controversial. Every one was right.

This project says yes to six protocols. Yes to offline mode. Yes to data import from Twitter, Instagram, Facebook, Reddit. Yes to desktop power-user features. Yes to community governance. Yes to BIP-39 key recovery. Yes to Tor. Yes to on-device ML translation.

You have a philosophy document that's 460 lines long. That's not a product. That's a manifesto.

**What's the one thing this app does that no other app does?**

If you can't answer that in one sentence, you don't have a product. You have a features list.

Let me answer it for you: "It's a social app that works offline." That's your one sentence. That's the one thing no one else does. Not six protocols. Not encryption indicators. Not protocol skins. **Offline-first social.**

But that's not the headline of your UX philosophy. It's buried on page seven. The heading says "Ownership by Default" and you talk about local data export and CSV formats. You're leading with the technical implementation instead of the magical experience.

Your one sentence should be: **"You can write a post in a tunnel."**

That's the story. That's the magic. That's the reason someone downloads this app instead of the other fifteen clients.

Everything else -- the protocols, the encryption, the identity linking -- that's engineering detail. It belongs in a README, not in the first conversation you have with a user.

---

**[He picks up the spec again. Holds it up.]**

This is brilliant work. The design principles are the best I've seen in this space. The anti-engagement patterns are courageous. The encryption UX is thoughtful. The protocol-aware design system shows talent.

But it's going to fail because you're trying to do too much at once. You're building a city when you should be building a room. A beautiful room. A room where one person can sit down, write their thoughts, and hit send. Even in a tunnel.

**Start there. Six protocols can wait. The empty room cannot. And have an opinion -- even if it makes people angry.**

If you don't offend anyone, you're not trying hard enough.

**[He drops the spec. Walks off stage.]**
