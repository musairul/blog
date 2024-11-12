---
layout: post
category: example2
---

This is about my experience at GreatUniHack 2024. Probably the most chaotic hackathon I've been apart of. And I've only been apart of 2 hackathons.

### 10:00 am - Team Building

We started the hackathon with 3 members. As we were trying to find a fourth, the 3rd member leaves. So now we need to find 2 more people to join our team. Great.

We found our third and fourth members and brainstormed ideas. The theme was "travelling through time". 

### 12:00 am - Hacking Begins

GUH is a 24 hour hackathon, starting at 12 and ending at 12.

Our first idea was to recreate the [minecraft AI thing](https://oasis.decart.ai/welcome) but where it works on a live video feed from your phone and changes your environment into different time periods which the user can select.

{% include image.html url="images/oasis_gameplay.png" description="Oasis Minecraft AI" %}

The minecraft AI thing is an entirely AI generated version of minecraft which responds to your inputs. So if you jump, it will AI generate Steve jumping in game.

As you can tell, this will probably be really hard to recreate ourselves, especially in 24 hours. So we scrapped the idea.

### 1:00pm - Idea 2: AR Interior Designer Through Time

So, with that idea scrapped and an hour wasted, we had to think of something else. We decided on a AR Interior Designer app that would look at your room and replace the furniture with different styles from different time periods - completely live. For example, the user could choose Victorian, and it would replace it with Victorian furniture. Cool. But actually, none of us know how to do AR. Okay then, lets reduce the scope to just a video of your room, and lets get rid of the AR. Next item on the menu, how are we going to generate the AI furniture?

Lucky for us, one of the sponsors 'Reply' provided a free OpenAI API key for the duration of the hackathon. The OpenAI API gives access to DALLE-2 - a text to image generation model. Well thats fine but we wanted to do image to image generation. Our plan was to generate a medieval version of every image frame and then stick it ontop. So our plans were foiled... right? No turns out google was just wrong and the API can infact to image to image generation. Great!

We got to work. To do image to image generation, the OpenAI API requires the input image, a transparent 'mask' over the input image to define what areas you want to generate over, and a prompt. 

That presented a few problems for us:

- How do we identify what the furniture are so we could prompt it to make an alternate version?
- How do we identify where the furniture are so we can create a mask over those areas?
- How do we generate the time period versions of that furniture?

### Nothing is ever as easy as it seems

Using my experience from previous computer vision projects, we could easily detect furniture using a pre-trained model online. Except I couldnt find any good models online. No matter, we could train it ourselves. But again, looking online there really werent any good datasets for it. They all focused on specific types of furniture like couches. The alternative then, was to use a more general object identification model. We chose to use YOLO, a model which generally identifies any objects in an image.

YOLO worked really well, identifying not just furniture but all the objects in the image. Great!

Onto the next issue, identifying *where* the furniture is. This was pretty easy, just grab the coordinates from whatever the YOLO model returned. Creating a mask was similarily easy, using some image manipulation libraries to cut out a rectangle over wherever the coordinates showed.

The third problem is similarily easy. Simply give the input image of a room, the mask, and a prompt saying: "create a {time period} version of {object} at the position {co-ordinates}" for every object. DALLE will then generate the image and provide a link to it which we can download and show to the user. 


After waiting around 30 seconds, we get the image link and download it. That took way too long, and it was clear that we would have to scrap the video idea and switch to just images instead.

Oh wow. The images were bad. Really bad. They literally had nothing to do with the prompt we gave, and would often times not even look like furniture at all. No amount of prompt engineering would fix it. I mean it was furniture but it really just wasnt the time period that we specified. It usually leaned to modern furniture.

### The Furniture AI Process

We start with an original image. [(source)](https://www.furniturechoice.co.uk/inspiration/modern-living-room-ideas_a10000058)

{% include image.html url="images/modernroom.png" description="" %}

We resize it to be square as DALLE can only deal with square images.

{% include image.html url="images/resized_modernroom.png" description="" %}

We create areas of transparency to define where DALLE should generate

{% include image.html url="images/furniture_transparent.png" description="Sofas and plants masked" %}

DALLE generates stuff in the masked areas based on the prompt.

{% include image.html url="images/alienroom.png" description="Output from the prompt 'Generate an Alien'" %}


As you can see, it did pretty bad. Instead of generating an alien, it generated a low quality human. But the worst part is it changed the sofas for no reason, the table, and removed the sofa on the left. Its unreliable and gave low quality generations. I expected it to generate an alien in each of the masked areas. I know here the prompt isnt really the same as victorian furniture for example, but I dont have access to the API key anymore and this was the only image I kept saved.


How do we solve this issue??

Well, we could use a better model instead of DALLE, like StableDiffusion or Flux. The problem was we didnt know how to self host these models and we didnt really have beefy laptops either. The OpenAI API key was given to us for free, we might as well take advantage of it right?

We thought for a while until we realised we could actually search for an image of the furniture on google, crop it, crudely stick it onto the image in the correct spot and then use DALLE to generate a *variation* of that image where the furniture actually looks like it belongs instead of just cropped and stuck ontop of the original image. Smart idea right? It really has potential!

So I got the searching furniture function working, and used OpenCV to detect the furniture and crop it out of the image, using other image processing libraries to stick it into the correct spot of the original image. Nice! We're almost there!

### Meanwhile...

When I initially proposed the furniture AI idea, half of the team thought it was boring. They decided to want to do a similar idea, but for fashion. It uses AI to generate what you would look like wearing different time period fashion items on a live video feed, while also detecting how much your current fit fits with other eras and styles of fashion. For example, your current fit is 70% hipster or something. It would be called "drip detector". Or "drip-thru-time". Or "drip to the future".

Okay, I will admit, those were banging names. I was convinced, but I continued working on my furniture idea anyway since I gathered the progress would easily transfer over to the fashion idea.

This meant however, only 2 people were working on the furniture AI and 2 people were working on the fashion AI. Seeing all the issues we faced with furniture AI, as well as the amazing name, we decided to go all in on "drip detector".

### 9:00pm - Drip Detector

Progress on drip detector looked promising. They had it use OpenCV to analyse a persons body and green screen it out, so we could place an outfit ontop of the green screen. We could potentially search for outfits on google and overlay them ontop, or use DALLE, and maybe since clothes are a lot more common it will generate them better.

We spent a lot of time on the furniture AI, but not all is lost, we can re-use a lot of functions from it!

We took a break before starting on drip detector and brainstormed more ideas.

"How about an app that uses computer vision to detect objects, and then use OpenAI's transcription feature to tell blind people what is in front of them and how far away and if its urgent to get out of the way or something? We could use Agentic AI to allow it to buy a shirt they see in the shops for example!"

"How about an app that allows us to talk to historical figures like Henry the 8th or something?"

"How about an app that detects fossils?"

This constant switching of ideas and not sticking to anything annoyed the teammate that was actually working on the drip detector so much, he straight up just left the group and went home. 

Lets be real. Why were we even brainstorming in the first place? We spent 8 hours on an app we scrapped and now we have nothing to show for it. We shouldve just stuck with drip detector from here, as we couldve used re-used functions we already made from the furniture AI, and added cool features like the agentic AI buying the time period accurate outfit it generates on you, for you.

Either way, for some reason we switched to a fossil detection app, which had a lot less functionality and ambition than any of our other ideas.

### 10:00pm - And then there were three...

From 9-10, we switched venues and now we scrapped everything we worked on up to this point and started our work on the fossil detector. We had a great name for it: Fossil-Eyes. (Get it? Fossil-*eyes*? Like fossilize? nvm...) Almost as good of a name as 'Drip Detector' tbh.

Anyway, with one man down and around 13 hours to go, we had to lock in. Unfortunately we were all really tired, but we had to work through the night in order to even have *any* project at all by the submission time. 

One teammate trained our own model on detecting fossils and finetuned it, resulting in an accuracy of 95%. The other created a frontend for the website while I made it so AI would provide facts depending on what the fossil is.


### 2:00am - LOCKED IN

By this point, my other team members were asleep. The frontend wasnt working, the app doesnt even fit the brief of "travelling through time", and time was running out. Frontend wasnt even working so I just got chatgpt to fix the bugs and add cool animations (AI was allowed in this hackathon). I integrated it with the fossil detection ML algorithm and the AI features. It worked surprisingly well. You could take a picture of a fossil and it would detect it very accurately!

{% include image.html url="images/fossileyes.png" description="Image of fossileyes analysing a fossil" %}

(The confidence value bug was fixed later on so it says 65% instead of 0.65 btw)

### 6:00am - What can go wrong, will go wrong.

Its morning, everyone was awake again and we had some sort of complete app. Right? Wrong. All of a sudden, the OpenAI API stopped working. WTH??? This was literally the worst time for this to happen.

Fine, forget the AI features. We realised in the morning that we dont qualify for literally any of the challenges set by the sponsors, or even the main theme of the hackthon. So thats what we tried to do.

I integrated Auth0 into the app while my teammate tried to integrate Firebase as a backend. We realised Firebase also has auth, so I deleted Auth0 from the app and focused on Firebase. Except firebase is the worst product I have ever used (as expected for a Google product). So we never ended up getting that to work. Whatever, we still have some sort of an app to show, and we can waffle about how it fits with "travelling through time". Something about "fossils being old and organic time capsules". Yeah, that sounds good.

### 10:00am - Morning motivation

2 HOURS TO SUBMISSION.

At this point, we were at the hackathon venue again. I asked the Reply team about the OpenAI API not working and somehow by literally just being there it fixed. So theres that.

With it fixed, it meant we could develop additional features. I downed an energy drink and got to work.

In the next 2 hours, I developed a map feature, where if a user discovers a fossil and takes a picture, it identifys it and adds it to a global map, where everyone can see the image you took, where you found it, the timestamp and your username. I also added a gallery feature, encouraging people to create a fossil collection. Things were looking good.

{% include image.html url="images/fossileyesmap.png" description="Terrible image of fossileyes map" %}

Not only that, but now we actually had a reason as to why this app fits the "travelling through time brief". As the map stored the timestamp, you could see how fossil distributions change over time due to factors like climate change. Our web app actually had real value now!

In hindsight, if we just focused our app on that specfic feature, adding like a cloropleth map and a year slider or something that wouldve been a way better idea. Oh well.

### 12:00pm - Submission

We submitted the app. Somehow, after all of that, we actually got something done and built a complete project and submitted it on time. Literally insane. We scrapped projects so many times, and worked on this like crazy throughout the night and actually delivered!

## Final Thoughts

Spoiler alert: We didnt win anything. For some reason we all had hope that we actually could but we didnt. Fair enough. In hindsight, we really shouldve just stuck to an idea from the beginning and took it all the way. I actually think if we had more time and stuck with the fossil idea we couldve developed it to be a winning project. Either way, the three of us still surviving demonstrated so much determination throughout the hackathon, and I really think no one else wouldve been this committed like we were.

GreatUniHack 2024 was an amazing experience, and one I hope to never experience again. Next hackathon, we're committing to an idea and sticking to it.

