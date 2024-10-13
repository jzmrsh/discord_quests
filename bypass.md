## Complete Recent Discord Quest

> [!NOTE]
> This no longer works in browser!
> 
> This no longer works if you're alone in vc! Somebody else has to join you!
>

> [!WARNING]
> There are now two quest types ("stream" and "play")! Pay attention to the instructions!
>

How to use this script:
1. Accept a quest under User Settings -> Gift Inventory
2. Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> to open DevTools
3. Go to the `Console` tab
4. Paste the following code and hit enter:
<details>
	<summary>Click to expand</summary>
	
```js
let wpRequire;
window.webpackChunkdiscord_app.push([[ Math.random() ], {}, (req) => { wpRequire = req; }]);

let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getStreamerActiveStreamMetadata).exports.Z;
let RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames).exports.ZP;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getQuest).exports.Z;
let ExperimentStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getGuildExperiments).exports.Z;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.flushWaitQueue).exports.Z;
let api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get).exports.tn;

let quest = [...QuestsStore.quests.values()].find(x => x.id !== "1248385850622869556" && x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now())
let isApp = navigator.userAgent.includes("Electron/")
if(!isApp) {
	console.log("This no longer works in browser. Use the desktop app!")
} else if(!quest) {
	console.log("You don't have any uncompleted quests!")
} else {
	const pid = Math.floor(Math.random() * 30000) + 1000
	
	let applicationId, applicationName, secondsNeeded, secondsDone, canPlay
	if(quest.config.configVersion === 1) {
		applicationId = quest.config.applicationId
		applicationName = quest.config.applicationName
		secondsNeeded = quest.config.streamDurationRequirementMinutes * 60
		secondsDone = quest.userStatus?.streamProgressSeconds ?? 0
		canPlay = quest.config.variants.includes(2)
	} else if(quest.config.configVersion === 2) {
		applicationId = quest.config.application.id
		applicationName = quest.config.application.name
		canPlay = ExperimentStore.getUserExperimentBucket("2024-04_quest_playtime_task") > 0 && quest.config.taskConfig.tasks["PLAY_ON_DESKTOP"]
		const taskName = canPlay ? "PLAY_ON_DESKTOP" : "STREAM_ON_DESKTOP"
		secondsNeeded = quest.config.taskConfig.tasks[taskName].target
		secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0
	}

	if(canPlay) {
		api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
			const appData = res.body[0]
			const exeName = appData.executables.find(x => x.os === "win32").name.replace(">","")
			
			const games = RunningGameStore.getRunningGames()
			const fakeGame = {
				cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
				exeName,
				exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
				hidden: false,
				isLauncher: false,
				id: applicationId,
				name: appData.name,
				pid: pid,
				pidPath: [pid],
				processName: appData.name,
				start: Date.now(),
			}
			games.push(fakeGame)
			FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [], added: [fakeGame], games: games})
			
			let fn = data => {
				let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value)
				console.log(`Quest progress: ${progress}/${secondsNeeded}`)
				
				if(progress >= secondsNeeded) {
					console.log("Quest completed!")
					
					const idx = games.indexOf(fakeGame)
					if(idx > -1) {
						games.splice(idx, 1)
						FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []})
					}
					FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
				}
			}
			FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
			
			console.log(`Spoofed your game to ${applicationName}. Wait for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
		})
	} else {
		let realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata
		ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
			id: applicationId,
			pid,
			sourceName: null
		})
		
		let fn = data => {
			let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value)
			console.log(`Quest progress: ${progress}/${secondsNeeded}`)
			
			if(progress >= secondsNeeded) {
				console.log("Quest completed!")
				
				ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc
				FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
			}
		}
		FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
		
		console.log(`Spoofed your stream to ${applicationName}. Stream any window in vc for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`)
		console.log("Remember that you need at least 1 other person to be in the vc!")
	}
}
```
</details>

5. Follow the printed instructions depending on what type of quest you have
    - If your quest says to "play" the game, you can just wait and do nothing
    - If your quest says to "stream" the game, join a vc with a friend or alt and stream any window
7. Wait for 15 minutes
8. You can now claim the reward in User Settings -> Gift Inventory!

You can track the progress by looking at the `Quest progress:` prints in the Console tab, or by reopening the Gift Inventory tab in settings.

## FAQ

**Q: Ctrl + Shift + I doesn't work**

A: Either download the [ptb client](https://discord.com/api/downloads/distributions/app/installers/latest?channel=ptb&platform=win&arch=x64), or use [this](https://www.reddit.com/r/discordapp/comments/sc61n3/comment/hu4fw5x/) to enable DevTools on stable


**Q: I get an error saying "Unauthorized"**

A: Discord has patched the script from working in browsers. Use the desktop app, or alternatively find some extension which lets you change your User-Agent and append the string `Electron/` anywhere in it.

They have also started checking how many people are in the vc, so make sure you join it on at least 1 other account.


**Q: I get a syntax error/unexpected token error**

A: Make sure your browser isn't auto-translating this website before copying the script. Turn off any translator extensions and try again.


**Q: I get a different error**

A: Make sure you're copy/pasting the script correctly and that you've have done all the steps.


**Q: Can you make the script auto accept the quest/reward?**

A: No. Both of those actions may show a captcha, so automating them is not a good idea. Just do the two clicks yourself.

___

<sup>Side note: Please don't post your "fixed" or "improved" versions of the script in the comments. It creates unnecessary confusion, often doesn't fix anything, and sometimes puts other people's accounts at risk. I might redact/delete such comments without notice. Thank you for understanding.</sup>
