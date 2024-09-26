# LC-script
How I had fun on lemoncloud minecraft server late 2023 - early 2024 and made 1600 lemons in just a couple months

Disclaimer: I probably didn't actually implement these in my gameplay, this is not an admission of guilt, all the following are hypothetical and may or may not actually work. Using these on a server will result in a ban if you are caught. 

The following is written for those with very rudimentary programming knowledge

## Why did I do this?
I like mc, but I don't like physically clicking on blocks/chests to sell drops or mine blocks. I like automation and I like to code small projects. Thus, I decided to go as over the top as possible in autmoating every step of playing survival. In the end I didn't have to open minecraft to play minecraft anymore. Enjoy the following probably fake story about how I made money on LC and saved my wrist from mindless clicking. I also liked lemons, I wanted a lot of them.

## Problems I needed to solve
1. To do anything with scripts is not allowed, so I needed to avoid detection. What I really needed was to see staff in vanish, which I semi-accomplished
2. On survival, I have big eman farms with over 125,000 double chests I needed to click on daily to sell the drops. I also needed to hide my ridiculous balance from /baltop
3. Everyone wants lemons, how do I get an advantage when it comes to buying them?

## My first failure
I tried using a macro + chest aura to sell chests. Basically recording my keystrokes moving around my farms selling chests then replaying it when I was afk. This failed when uni teleported me a few blocks away from my farm and I kept moving and clicking as I was afk. This also was not a good method because my computer had to be physically open and I had to manually run the macro recording and leave my computer be for the hours it took to sell my farms. Props to uni for getting banning me for 30 days, it was well deserved.

I wanted a method to sell my farms without minecraft even open on my laptop, and I want it to run automatically without any manual input at all. I also wanted my bot to be aware of the world around it and detect force teleports and if a mod messed with my inventory mid sell e.g remove my sell wand from my hand. This was all impossible with a simple macro.

## The solution
Free amazon EC2 servers and [Mineflayer.](https://github.com/PrismarineJS/mineflayer) EC2 servers let me host my bots on amazon servers and I could control them through a command line, and eventually a discord server. Mineflayer let me write bots that do 99% of what a real player can do in the game through a script.

### Semi vanish exploit - Logging staff activity
Since I was already banned, I knew I was under scrutiny when I came back. So with a mineflayer bot I decided to log all staff activity 24/7, even when they are in vanish. Unfortunately vanish is a great plugin that completely hides staff packets from clients (us players). Thus there is no way to actually detect staff in vanish as the server doesn't even send us those packets. However it seems that /seen will log the staff joining a specific gamemode, even if they join while in /v. Even though /seen shows them as 'Offline' it logs their last seen moment as the moment they joined the specific gamemode in /v. 


This information by itself isn't useful, but if logged consistently evey hour of every day you can learn helpful trends e.g hours of the day where there is little staff activity. Obtaining this information meant a bot switching between every gamemode, running /seen on every staff memeber, and logging it - every hour of every day. I collected a couple months of information like this. Remember, this bot is hosted on a virtual server with a timer to wake up every hour, so my computer does not have to be on for this to happen. This made for some very interesting graphs...

<img width="1112" alt="Screenshot 2024-09-25 at 12 11 04 AM" src="https://github.com/user-attachments/assets/194d8bf9-a100-4a2a-bfe5-ebd82bea2a32">

This is the culminative heat map of the data collected over 108 days. The X-axis shows the hours of each day from 0-24. The number in the boxes show how many times a specific staff was seen at that hour, ignoring what day of the week it was. Darker colored squares meant at that hour, it is more likely to find that particular staff online. Hovering over the box shows the exact dates that they were seen. Of course, with this data we can represent it in many forms. I do have to say, as much as people give staff shit for not moderating, they do a good job getting online and trying their best <3

<img width="724" alt="Screenshot 2024-09-25 at 12 16 29 AM" src="https://github.com/user-attachments/assets/68c0e5cb-8d6e-4e0f-8df1-eddf4b48461a">

For example here we can see a specific mod's hourly activity in a specificed time range. We can clearly see from 2am to 9am they are never online. Beautiful time to run a bot right? Or not, remember I probably didn't actually do all this, this is all fiction. 

<img width="836" alt="Screenshot 2024-09-25 at 12 28 18 AM" src="https://github.com/user-attachments/assets/dfa1b2a8-2228-4312-b614-e80a1c50772c">

Here we can clearly see how this method semi solves the vanish issue, we can see small gaps in the staff activity, usually for 1-2 hours. We can assume the staff was in vanish at these times if we were to err on the side of caution.

### Selling chests automatically on survival
Well now I have a good idea on the best time to run the bot with the smallest chance of detection. Time to write a bot that flys around my chests and auto sells them. This part is boring, but a couple things to highlight. My chest collections were in a 3 adjacent 64x64 walls. I had 10 farms, so that totals to 245,760 chests. If any OGs on survival remember me mass buying chests and hoppers for ridiculous prices, this is why. Of course, I didn't manually build these farms by hand, I had a bot that built farms as well. Anyways, I digress. The biggest issue was avoiding the same mistake that got me banned in the past. If I was ever force teleported, I wanted to be able to detect that and stop my bot. Luckily Mineflayer has a 'force-teleport' function that does just this, however TPS drops sometimes triggered this, so the majority of the time I ran the bots without this protection. Risky for sure, but again I knew staff was offline :D. Of course, this ran automatically, on a virtual server, everyday at a scheduled time by itself.

Below are some snippets of the main functions that flys through my wall of chests, clicking on them with a sell wand to sell the contents. Remember this code probably doesn't work as I probably never actually botted on the server.
```js
// Handles polarity when moving along different axes
function mapper(ax,direction,flipflop,chestFacing){
  
  //0 : initial position offset
  //1 : move direction
  //2 : initial chest offset for for loop counting
  output = []
  if (ax == 'x'){
    output.push(v(0,-1,-1chestFacing*2))
    output.push(v(flipflop*direction,0,0))
    output.push(v(-1*direction,0,0))
  }else{
    output.push(v(-1*chestFacing*2,-1,0))
    output.push(v(0,0,flipflop*direction))
    output.push(v(0,0,-1*direction))
  }
  return output

}
// Handles selling a single 64x64 wall with inputs denoting the axis the wall lies on and direction of chests
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

// Handles moving between each 64x64 wall of chests
const getToNewLoc = async (oldChestWallAx,oldChestWallDirection,newFirstChestCoord,newChestFacing) => {
  startedEvent = true
  var currentPlayerLocation = bot.entity.position
  var newChestStartingLoc = newFirstChestCoord
  if (oldChestWallAx == 'x'){
    mapped = mapper('y',0,0,newChestFacing)
    newChestStartingLoc = newFirstChestCoord.offset(mapped[0].x,mapped[0].y,mapped[0].z)
  }else{
    mapped = mapper('x',0,0,newChestFacing)
    newChestStartingLoc = newFirstChestCoord.offset(mapped[0].x,mapped[0].y,mapped[0].z)
  }}

// Driver for selling chests, combines the other functions to sell 3 walls of chests among 10 different farms
const completeSell = async () => {
  bot.chat('/fly on')
  deposit = 'w'
  bot.chat('/pv 5')
  await bot.waitForTicks(40)
  // await useSellMulti()
  // await bot.waitForTicks(40)
  // await useSellMulti()

  await bot.waitForTicks(40)
  await equipSellWand()
  for (let i = 1; i < 11;i++){
    if (!moved){
      var currentHome = i
      const datee = new Date()
      console.log(datee.toString())
      console.log('Now selling farm ' + currentHome.toString())
      await bot.chat('/home farmsell' + currentHome.toString())
      
      await bot.waitForTicks(70)
      await sellChests(localeFirstWallCoords[i],localeFirstWallAx[i],localeFirstWallDirection[i],localeFirstChestDirection[i],16,68)
      isBigFlying = true
      await getToNewLoc(localeFirstWallAx[i],localeFirstWallDirection[i],localeSecondWallCoords[i],reverser(localeFirstWallDirection[i]))
      isBigFlying = false
      // The second walls wall direction (sell direction) is the same as the first wall chest facing direction
      await sellChests(localeSecondWallCoords[i],reverser(localeFirstWallAx[i]),localeFirstChestDirection[i],reverser(localeFirstWallDirection[i]),16,68)
      isBigFlying = true
      await getToNewLoc(reverser(localeFirstWallAx[i]),localeFirstChestDirection[i],localeThirdWallCoords[i],reverser(localeFirstChestDirection[i]))
      isBigFlying = false
      await sellChests(localeThirdWallCoords[i],localeFirstWallAx[i],reverser(localeFirstWallDirection[i]),reverser(localeFirstChestDirection[i]),16,68)
      console.log('Done with farm ' + currentHome.toString())

    }
    
  }}
```
### Alt Bank system
At my peak I netted around 6-7 billion from selling epearls a day without even opening my laptop. This is a problem. If I am constantly on /baltop, people will notice. To get around this, I had my bots automatically distrubute the money among many alts, so I would stay off /baltop, easy! Or so I thought. It was a pain to manually manage the balance of all my alts. Sometimes alts would fail and go offline. This meant when the money was distrubuted, some alts would not recieve their share, causing other alts to recieve too much money and peak through on /baltop.

The solution? A self-balancing alt network. Whenever money was sent to an alt, the entire alt network would send each other money so that all the accounts ended up with the same balance. Then when I would want to withdrawl money from the alt network (to my main account), they would each send me a portion of the amount of money I requested. For example, if I had 10 alts and I wanted to withdrawl 1B, each alt would send me 100M. Of course, this was done automatically so I wouldn't have to log into each account and manually type /pay.

Here's a little snippet of that code for your own edification
```js
// Function that actually runs /pay in game with the correct usernames and amounts
function pay(senderIndex,recieverIndex, amt){
	console.log(usernames[senderIndex] + ' payed ' + usernames[recieverIndex] + ' ' + amt)
	let cmdString = ''
	if (senderIndex == 9){
		bot.chat('/pay ' + usernames[recieverIndex] + ' ' + amt)
	}else{
		cmdString = ecs[accounts[senderIndex][1]] + '"screen -S ' + accounts[senderIndex][0] + " -X stuff '/pay " + usernames[recieverIndex] + ' ' + amt  + "\\" + "n" + "'" + '"'
		// console.log(cmdString)
		child.exec(cmdString)
	}
}
// Helper function for balanceBal()
function removeIndex(array,indexArray){
	let newArr = []
	for (let i=0; i<indexArray.length; i++){
		array[i] = -1
	}
	for (let i=0; i<array.length; i++){
		if (array[i] != -1){
			newArr.push(array[i])
		}
	}
	return newArr
}

// Balances the balance of alts
function balanceBal(){

	let higherArray = []
	let lowerArray = []
	let doneArray = []
	let total = 0
	for (let i = 0; i < bals.length; i++){
		total += bals[i]
	}

	let targetBal = total/bals.length
	console.log('Target Bal: ' + targetBal)

	for (let i = 0; i < bals.length; i++){
		if (bals[i] > targetBal){
			higherArray.push(i)

		}
		else if (bals[i] < targetBal){
			lowerArray.push(i)
		}
		else{
			doneArray.push(i)
		}
	}

	for (let i = 0; i < higherArray.length; i++){
		let indexToSplice = []
		let difference = bals[higherArray[i]] - targetBal
		
		bals[higherArray[i]] -= difference
		for (let j = 0; j < lowerArray.length; j++){
			
			if (difference == 0){
				break
			}

			if (bals[lowerArray[j]]+difference < targetBal){
				pay(higherArray[i],lowerArray[j],Math.round(difference))
				bals[lowerArray[j]]+= difference
				
				difference = 0
			} else{
				difference = difference - (targetBal - bals[lowerArray[j]])
				pay(higherArray[i],lowerArray[j],Math.round((targetBal - bals[lowerArray[j]])))
				bals[lowerArray[j]]+= (targetBal - bals[lowerArray[j]])


				if (Math.abs(bals[lowerArray[j]] - targetBal) < 1){
					indexToSplice.push(j)
				}
				
			}

		}
		// console.log(indexToSplice)
		lowerArray = removeIndex(lowerArray,indexToSplice)
	}

}
```

### Converting money to lemons - discord bot
Finally it is time to use that in-game-money to buy lemons from other players. I didn't want to sit online all day monitering chat waiting for people to sell lemons for in-game-money. So, I wrote a discord bot that notified me when someone mentioned 'lemon' in the game chat. I had to filter out 'ilemon' or 'lemoncloud', but for the most part when I got pinged, it was a potential sale. My discord bot could also interact with in game chat, so I could buy lemons and interact with players through my phone. I was very close to implementing ChatGPT to automaically contact players and broker sales, but I didn't want to pay OpenAI to use their API.

![CleanShot 2024-09-25 at 01 27 25@2x](https://github.com/user-attachments/assets/836a6314-a11e-40e5-8412-40de2fd57a1a)

This discord bot was proudly made with [discordjs](https://discordjs.guide/#before-you-begin) and [Mineflayer](https://github.com/PrismarineJS/mineflayer). Pretty quickly I got to a total of around 1600 lemons before my days of doing nothing ran out. Anyways, I ended up selling some of those lemons for some real life money, the final goal of this project. Or did I? I guess we will never really know.

Again, all the above is probably fiction, who would spend so much time and effort getting fake currency?

Until next time,
<br>
blessed



<br>
<br>
<br>


A small thank you to steel for letting me use your server to test my bots before deploying them <3

