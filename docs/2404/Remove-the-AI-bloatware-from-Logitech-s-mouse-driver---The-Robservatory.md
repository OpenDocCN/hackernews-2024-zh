<!--yml
category: 未分类
date: 2024-05-27 13:34:19
-->

# Remove the AI bloatware from Logitech’s mouse driver – The Robservatory

> 来源：[https://robservatory.com/remove-the-ai-bloatware-from-logitechs-mouse-driver/](https://robservatory.com/remove-the-ai-bloatware-from-logitechs-mouse-driver/)

I absolutely love Logitech's Mac [MX Keys keyboard](https://robservatory.com/review-logitech-mx-keys-for-mac/) and [MX Master mouse](https://robservatory.com/review-logitech-mx-master-2s-mouse/) (though I've now updated to version 3 of the mouse). And generally, their software has been pretty good, too.

But a recent update added an "AI Prompt generator" feature to the mouse side of things, which is absolute garbage—I'm not saying it's bad, as I've never tried it. It's garbage as in there's no reason my mouse needs an AI prompt generator connected to a button. Even worse, as [Stephen Hackett discovered](https://512pixels.net/2024/04/ai-overlay-tmp-home-folder-mac-os/), it creates a folder (at the top level of your home folder, no less!) with the ugly name of `ai_overlay_tmp`.

Thankfully, when Stephen [posted about this](https://mastodon.social/@ismh@eworld.social/112320597255841194) on Mastodon, user @flipneus [posted the solution](https://mastodon.social/@flipneus/112321297224942175). And in case that post ever goes away, here it is:

In Finder, open the top-level Library → Application Support folder, then navigate to Logitech → LogiOptionsPlus, and open *app_permission.json* in your favorite pure text editor. Add a comma after the last `}` on the line *before* the final `}`, then add these lines:

```
 "aipromptbuilder": {
  "value": false
 }
}
```

When done, the end of the file should look like this (though the commands in yours may differ):

```
...
  },
 "backlight": {
  "value": true
 },
 "aipromptbuilder": {
  "value": false
 }
}
```

The important part is the added comma after (in my file) the backlight-related section. Save the file when done editing, and reboot.

After the reboot, you can delete the `ai_overlay_tmp` folder—and there won't be an AI generator option in the Logi Options+ app any more. (Alternatively, Stephen points out you can use [SteerMouse](https://plentycom.jp/en/steermouse/index.html) to program the buttons on the Logitech.)

Thank you, Stephen and @flipneus!