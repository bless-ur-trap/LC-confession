# LC-script
How I had fun on lemoncloud minecraft server late 2023 - early 2024 and made 1600 lemons in just a couple months

Disclaimer: I probably didn't actually implement these in my gameplay, this is not an admission of guilt, all the following are hypothetical and may or may not actually work. Using these on a server will result in a ban if you are caught. 

The following is written for those with very rudimentary programming knowledge

## Why did I do this?
I like mc, but I don't like physically clicking on blocks/chests to sell drops or mine blocks. I like automation and I like to code small projects. Enjoy the following probably fake story about how I made money on LC and saved my wrist from mindless clicking. I also liked lemons, I wanted a lot of them.

## Problems I needed to solve
1. To do anything with scripts is not allowed, so I needed to avoid detection. What I really needed was to see staff in vanish, which I semi-accomplished*
2. On survival, I have big eman farms with over 125,000 double chests I needed to click on daily to sell the drops. I also needed to hide my ridiculous balance from /baltop
3. Everyone wants lemons, how do I get an advantage when it comes to buying them?

## My first failure
I tried using a macro + chest aura to sell chests. Basically recording my keystrokes moving around my farms selling chests then replaying it when I was afk. This failed when uni teleported me a few blocks away from my farm and I kept moving and clicking as I was afk. This also was not a good method because my computer had to be physically open and I had to manually run the macro recording and leave my computer be for the hours it took to sell my farms. Props to uni for getting banning me for 30 days, it was well deserved.

I wanted a method to sell my farms without minecraft even open on my laptop, and I want it to run automatically without any manual input at all. I also wanted my bot to be aware of the world around it and detect force teleports and if a mod messed with my inventory mid sell e.g remove my sell wand from my hand. This was all impossible with a simple macro.

## The solution
Free amazon EC2 servers and [Mineflayer.](https://github.com/PrismarineJS/mineflayer) EC2 servers let me host my bots on amazon servers and I could control them through a command line, and eventually a discord server. Mineflayer let me write bots that do 99% of what a real player can do in the game through a script.

### Semi vanish exploit - Logging staff activity
Since I was already banned, I knew I was under scrutiny when I came back. So with a mineflayer bot I decided to log all staff activity 24/7, even when they are in vanish. Unfortunately vanish is a great plugin that completely hides staff packets from clients (us players). Thus there is no way to actually detect staff in vanish as the server doesn't even send us those packets. However it seems that /seen will log the staff joining a specific gamemode, even if they join while in /v. Even though /seen shows them as 'Offline' it logs their last seen moment as the moment they joined the specific gamemode in /v. This information by itself isn't useful, but if logged consistently evey hour of every day you can learn helpful information. This meant a bot switching between every gamemode, running /seen on every staff memeber, and logging it - every hour of every day. I collected a couple months of information like this. Remember, this bot is hosted on a virtual server, so my computer does not have to be on for this to happen. This made for some very interesting graphs...

<img width="1112" alt="Screenshot 2024-09-25 at 12 11 04 AM" src="https://github.com/user-attachments/assets/194d8bf9-a100-4a2a-bfe5-ebd82bea2a32">

This is the culminative data over 108 days of data collection. The X-axis shows the hours of each day from 0-24. The number in the boxes show how many times a specific staff was seen at that hour, ignoring what day of the week it was. Hovering over the box shows the exact dates that they were seen. Of course, with this data we can represent it in many forms. I do have to say, as much as people give staff shit for not moderating, they do a good job getting online and trying their best <3

<img width="724" alt="Screenshot 2024-09-25 at 12 16 29 AM" src="https://github.com/user-attachments/assets/68c0e5cb-8d6e-4e0f-8df1-eddf4b48461a">

For example here we can see a specific mod's hourly activity in a specificed time range. We can clearly see from 2am to 9am they are never online. Beautiful time to run a bot right? Or not, remember I probably didn't actually do all this, this is all fiction. 

<img width="836" alt="Screenshot 2024-09-25 at 12 28 18 AM" src="https://github.com/user-attachments/assets/dfa1b2a8-2228-4312-b614-e80a1c50772c">

Here we can clearly see how this method semi solves the vanish issue, we can see small gaps in the staff activity, usually for 1-2 hours. We can assume the staff was in vanish at these times if we were to err on the side of caution.

### Selling chests automatically on survival
Well now I have a good idea on the best time to run the bot with the smallest chance of detection. Time to write a bot that flys around my chests and auto sells them. This part is boring, but a couple things to highlight. My chest collections were in a 3 adjacent 64x64 walls. I had 10 farms, so that totals to 245,760 chests. If any OGs on survival remember me mass buying chests and hoppers for ridiculous prices, this is why. Of course, I didn't manually build these farms by hand, I had a bot that built farms as well. Anyways, I digress. The biggest issue was avoiding the same mistake that got me banned in the past. If I was ever force teleported, I wanted to be able to detect that and stop my bot. Luckily Mineflayer has a 'force-teleport' function that does just this, however TPS drops sometimes triggered this, so the majority of the time I ran the bots without this protection. Risky for sure, but again I knew staff was offline :D. Of course, this ran automatically, on a virtual server, everyday by itself.

At my peak I netted around 6-7 billion igm a day without even opening my laptop. This is a problem. If I am constantly on /baltop, people will notice. To get around this, I had my bots automatically distrubute the money among many alts, so I would stay off /baltop, easy! I would then buy lemons to lower my balance.

Below is a snippet of the main function that flys through my wall of chests, clicking on them with a sell wand to sell the contents
```js
const sellChests = async (startingChestPOS,ax,direction,chestFacing,rows,cols) => {
  startedEvent = true
  currentChest = startingChestPOS
  var flipFlop = 1
  // x and y are for player model to have room to sell
  var startingLoc = startingChestPOS.offset(mapper(ax,direction,flipFlop,chestFacing)[0].x,
    mapper(ax,direction,flipFlop,chestFacing)[0].y,
    mapper(ax,direction,flipFlop,chestFacing)[0].z)
  
  console.log('starting at' + startingLoc.toString())
  // z-shift is since the first step of for loop is to shift +1
  currentChest = currentChest.offset(mapper(ax,direction,flipFlop,chestFacing)[2].x,
    mapper(ax,direction,flipFlop,chestFacing)[2].y,
    mapper(ax,direction,flipFlop,chestFacing)[2].z)

  isBigFlying = true
  await flyToDest(startingLoc,true)
  isBigFlying = false

  for (let j = 0; j < rows;j++){
    console.log('\nRow: ' + j.toString())
    console.log('Col: ')
    for (let i=0; i<cols;i++){
      if (moved){
        break
      }
      
      currentChest = currentChest.offset(mapper(ax,direction,flipFlop,chestFacing)[1].x,
        mapper(ax,direction,flipFlop,chestFacing)[1].y,
        mapper(ax,direction,flipFlop,chestFacing)[1].z)
      var chesty = bot.blockAt(currentChest)
      openChest(chesty)
      process.stdout.write(i.toString() + ' ')
      
      if (i != cols-1){
        startingLoc = startingLoc.offset(mapper(ax,direction,flipFlop,chestFacing)[1].x,mapper(ax,direction,flipFlop,chestFacing)[1].y,mapper(ax,direction,flipFlop,chestFacing)[1].z)
        await flyToDest(startingLoc)
      }
    }
    if (moved){
      break
    }
    flipFlop = flipFlop * -1
    startingLoc = startingLoc.offset(0,1,0)
    await flyToDest(startingLoc)
    currentChest = currentChest.offset(0,1,0)
  }
  startedEvent = false
}
```

### Converting money to lemons - discord bot
I didn't want to sit online all day monitering chat waiting for people to sell lemons. So, I wrote a discord bot that notified me when someone mentioned 'lemon' in the game chat. I had to filter out 'ilemon' or 'lemoncloud', but for the most part when I got pinged, it was a potential sale. My discord bot could also interact with in game chat, so I could buy lemons and interact with players through my phone. 

![CleanShot 2024-09-25 at 01 27 25@2x](https://github.com/user-attachments/assets/836a6314-a11e-40e5-8412-40de2fd57a1a)

This discord bot was proudly made with [discordjs](https://discordjs.guide/#before-you-begin) and [Mineflayer](https://github.com/PrismarineJS/mineflayer). Pretty quickly I got to a total of around 1600 lemons before my days of doing nothing ran out. Anyways, I ended up selling those lemons for some real life money, the final goal of this project. Or did I? I guess we will never really know.

Again, all the above is probably fiction, who would spend so much time and effort getting fake currency?

Until next time
<br>
bless



<br>
<br>
<br>


A small thank you to steel for letting me use your server to test my bots before deploying them <3

