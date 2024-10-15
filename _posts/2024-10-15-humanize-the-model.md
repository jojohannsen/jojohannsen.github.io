---
title: "Humanize the Model"
date: 2024-10-15
categories: agents
---
It's quiet here in Joshua Tree.  The days have 36 hours.  

The picture you've imagined is human.  Something humans like to process.

```python
result = human.invoke("It's quiet here in Joshua Tree.  The days have 36 hours.")
```

What data type is `result`?

It depends on how you've configured your human.  One way to configure would be:

```python
# Extract this structured data
class PhysicalAddress(BaseModel):
  name: str = Field(..., description="Name of the place")
  city_name: Optional[str] = Field(None, description="City Name, or Nearest City")
```

Then we configure our human:

```python
human = ChatOpenAI(model="gpt-4o").with_structured_output(PhysicalAddress)
```

So one would expect, after invoking the human, that there would be a memory containing a PhysicalAddress with name "Joshua Tree", and maybe a city name of "Joshua Tree".

But really a human is so much more than that.  Humans have Feelings, too, and that is important.

```python
class Feelings(BaseModel):
  happy: bool = Field(..., description="Tone of message is happy, or brings back happy memories")
  sincere: bool = Field(..., description="Tone of message is sincere")
  sad: bool = Field(..., description="Tone of message is sad, or brings back sad memories")
```

Now that we have designed our Feelings, we can configure our human.

```python
human_with_feelings = ChatOpenAI(model="gpt-4o").with_structured_output(Feelings)
```

But we really can't create lots of different models for the same human.  It would be much better to just make the Structured Data handle all the information.

```python
class HumanSoFar(BaseModel):
  feelings: Feelings
  address: PhysicalAddress
  
super_realistic_human = ChatOpenAI(model="gpt-4o").with_structured_output(HumanSoFar)
```

Invoking the above 'super_realistic_human' gives:
```python
HumanSoFar(feelings=Feelings(happy=True, sincere=True, sad=False), address=PhysicalAddress(name='Joshua Tree', city_name=None))
```

---

Something like [Honcho](https://honcho.dev/) has a much more advanced model and implementation, so would be much more human-like.

Also, [trustcall](https://www.youtube.com/watch?v=-H4s0jQi-QY) allows more complex models to be populated more reliably.   The above example isn't complicated enough to need it.
