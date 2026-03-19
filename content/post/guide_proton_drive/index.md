---
title: "From Discord to Drive: Securing Your Images"
description: Why your "private" Discord uploads aren't actually private.  Here's how to fix it in 9 minutes
date: 2026-03-19T01:49:26-04:00
image: 
math: 
license: 
comments: false
draft: true
build:
    list: always    # Change to "never" to hide the page from the list
tags: [
    "privacy",
    "security",
    "end-user"
]
categories: [
    "guide"
]
---

Let's be honest... 

We all have files we'd rather not have floating around on someone else's servers. Tax returns, medical records, legal documents, or yes, even personal photos. The problem is, most of us have gotten used to treating cloud storage services like a magical black box where we assume everything just stays safe.

Here's the thing: it usually doesn't. 

## The Problem With Mainstream Cloud Services

Before we dive into how to use Proton Drive properly, let's talk about why you might want to think twice about using Google Drive, Dropbox, Discord, or similar services for sensitive files.

Cloud storage isn’t a magical black box.  Cloud storage is basically a fancy way of saying: someone else’s computer.  There is some extra software that supposedly protects your files, but how can we be sure?  What if this other person has different priorities than you? What if they want to use your files to train AI, send you ads, or to make sure you’re not uploading copyrighted content?

All of a sudden your files don’t seem so secure after all.

### What Actually Happens to Your Files

When you upload to most mainstream services:

1. **They hold the encryption keys** — The companies claim your files are encrypted, which they are, but the company holds the keys. This means they can access your data. 
2. **Content scanning** — Many companies scan uploaded files for copyright violations, policy enforcement, or advertising purposes. Google Drive, for example, has historically scanned emails and files for various automated systems. They make money from advertising after all! 
3. **Government requests** — These companies regularly receive subpoenas and data requests. They must comply with many of them, giving access to your files to the government(s) in the countries where their servers are located.

If the company maintains access to your files, this is also a vector for malicious actors to gain access to your files too.  Therefore, it’s not just the cloud storage service you need to trust, you need to trust the people who may gain access to your files maliciously.  That could be anyone!

## Case Study: Discord

Discord is convenient, exciting, and where many younger people organize their social lives.  Gone are the days of Skype, Google Meet, Instagram group chats, Facebook Messenger, Snapchat, or SMS.  Discord has become a one size fits all for a lot of people.  And for good reason!  It’s convenient, and offers great features for the social lives of young people.  

Discord is particularly worth calling out because it's so commonly used for file sharing, especially among younger generations. Yet, its architecture creates many privacy risks.

### How Discord Hosts Images

When you upload an image to Discord, whether in a private DM, private server, or a public channel, it gets stored on Discord's public Content Delivery Network (CDN). This means:

- **Public URLs** — Every uploaded file receives a unique URL that anyone can access if they obtain it.
- **No access controls at the CDN level** — The CDN doesn't check whether you're authorized to view the file; it just serves it to whoever has the link.
- **Permanent links** — Until recently, these links didn't expire, meaning files could be accessed indefinitely.

This makes total sense from Discord’s point of view.  It’s a public messaging platform after all.  It’s main goals are convenient and quick access for social groups.  But when people start sharing nude pictures, medical documents, tax documents, or anything else you may deem sensitive, this starts to become a problem.

### Try it for Yourself!

1. Go into a private message where you have sent an image,
2. Right click the image,
3. Select “Copy Link”
4. Paste that link in your web browser, and see what comes up!

**IMAGE PLACEHOLDER**

That’s your picture!  Anyone can see it.  For sharing images, this is worse than the Snapchat privacy concerns ever were.

### Third Party Access Problem

This architecture creates several vulnerabilities:
1. **Link leakage** — If someone accidentally shares a link outside the intended channel, or if a link gets scraped from logs, cached pages, or forwarded messages, the file becomes publicly accessible!  These are known vulnerabilities that bad actors are actively taking advantage of.
2. **Cache** — Third-party CDNs like Cloudflare may cache images to reduce load on Discord's servers. Even if Discord deletes the original file, cached copies can persist on these third-party servers.  These CDNs are used to make images faster to view, but can cause a privacy concern!
3. **Predictable URL patterns** — Discord's content URLs follow predictable patterns based on channel and attachment IDs. This means that with enough knowledge, someone could potentially find and access files they weren't meant to see.
4. **No Central Access Controls** — You can delete images that you send to prevent them from being seen, but what happens if you sent it to multiple places?  Or you sent it a long time ago and now forget where?  In order to delete the image, you have to go and find every instance of it and delete it.  You could be missing more than you think!

### Bottom Line

While Discord is constantly trying to improve their security, and new features are tried and tested regularly, it’s clear that there is a conflict.  Discord’s main purpose is to be a fast, convenient, social messaging platform for the masses.  They want quick access to images, messages, and easy viewing.  This is often times in conflict with the need for privacy and security.

Most users reasonably assume their attachments are somewhat secure, especially when shared in private messages. That assumption couldn’t be more incorrect.

# Enter Proton Drive

Proton Drive flips the script on cloud storage, focusing on privacy first.  Here’s what makes it different:
- **End-to-End Encryption (E2EE)** — Files are encrypted on your device, before they are sent to Proton Drive.
- **Zero-Access Architecture** — Proton can’t see, scan, audit, your files.  They literally don’t have the encryption keys to decrypt them.
- Based in Switzerland — Proton is based in Switzerland, which has much stronger privacy protections than the United States.  
- **Open Source** — Proton software is open source.  They are regularly audited by third party privacy and security watchdogs to ensure their software is as secure as they say it is.  You can take a [look yourself](https://github.com/ProtonDriveApps) too!

When you upload a file on Proton, you don’t need to trust that their interests are aligned with yours because the architecture itself prevents them from viewing your files. If they tried to look at your files, they would find a bunch of unintelligible ones and zeros.

You can read more about Proton Drive’s security model [here](https://proton.me/blog/protondrive-security), and more about how your encryption keys are stored [here](https://proton.me/support/how-is-the-private-key-stored).

## Getting Started

If you don’t have an account yet, head to [proton.me/drive](https://proton.me/drive) and sign up.  As of writing, the free tier gives you 5GB of storage.  This is more than enough to start securing some sensitive documents or pictures.

### Step 1: Upload Your Files

1. Open [Proton Drive](https://drive.proton.me/) in your browser, or download the mobile app (both iOS and Android)
2. Click the **Upload** button, or drag your files directly into the window
3. Wait for the upload to finish.  You should see a green checkmark when it’s done!

Tip: You can upload entire folders, not just individual files.

**IMAGE PLACEHOLDER**

### Step 2: Create Shareable Links

1. Right click the file you want to share (or tap the three dots on the right side of the file on mobile)
2. Select S**hare** (or **Manage Access** if you are on mobile)
3. Activate the **Public Link** toggle (or the **Share with Anyone** toggle on mobile)
4. Copy the link!

Here is where things get interesting…

### Step 3: Locking Down Your Links

By default, Proton Drive links are private.  However there are many options for you to add extra layers of protection.

#### Password Protection

This is a non-negotiable for sensitive files.
- Click “Set password or expiration date”
- Toggle on “Require password”
- Create a password
- Click “Save changes”

This will require anyone with the link to also enter a password in order to view the file.  Just having a password alone is a big improvement for privacy since the link is useless unless the person also has the password.  You can ensure it’s even safer by:

- Creating a strong password.  Think 12+ characters with numbers, letters, and symbols.
- Send the password in a different channel than the link – This makes it more difficult to find both the link and the password, and to propagate it to unwanted third parties.

#### Expiration Dates

Be realistic about how long access allowed for.  If you’re sending a picture to a discord channel, chances are there’s no need for it to be shared forever.  An expiration time of a week will make it so that your images will stop being shared, and your privacy is maintained without you having to think about it.
- Click “Set password or expiration date”
- Toggle on “Expiry”
- Select a date when this link will expire
- Click “Save changes”

### Managing Shares

Even if you don’t set an expiry date, managing your shares through Proton Drive is a lot easier than scouring through discord messages.  You can view all your active shares at any time.
- On the sidebar, click the “Shared” tab.

This will allow you to view all your currently shared files and images.  It’s a good idea to occasionally take a look and decide whether you want to remove any.

### Taking Things to the Next Level

Sharing files through links can still be risky, even if it is much more secure than otherwise.  There’s always a chance it can fall into the wrong hands.  The most secure way to share sensitive files in Proton Drive, is to share it ONLY to other Proton Drive users.  This requires the people you want to share with to have Proton accounts.  This way, you can ensure that no unauthorized third parties will be able to view your sensitive shares.
- In this instance, the weak link is the trust between you and the person you are sharing with. They could screenshot, or show your sensitive files to other people! 

# Wrapping Up

Look, I get it.  Setting up secure sharing takes some extra time, it’s not as convenient compared to dumping a file in a discord.  But those extra seconds matter when you’re dealing with documents and images that could impact your financial security, personal safety, and reputation.  

Proton Drive isn’t perfect.  At the end of the day, you, as the individual are also a weak link in protecting your privacy.  That will always be the case.  However the architecture helps remove a lot of other variables to consider.  It’s built with the right priorities, and you don’t need to trust Proton in order to trust that your files are safe.

This guide focused on Proton, but it’s really the effort that counts.  Proton is my choice, but you may have different preferences.  That’s okay.  Making an effort puts you head an shoulders above the default choices out there.

**Stay safe and private!**