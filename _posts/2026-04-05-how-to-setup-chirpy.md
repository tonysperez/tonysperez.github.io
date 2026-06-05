---
title: A Non-Developers Guide to Building This Website
description: A step-by-step guide for setting up Jekyll on GitHub Pages
categories: [guide]
tags: [chirpy, jekyll, github, github pages]
---

This website is hosted on GitHub, for free. Not that the files are on GitHub for some CI/CD pipeline to pull to a web server somewhere, but this website is hosted on GitHub's servers. On top of that, this website is entirely code-based. So posts, themeing, everything, can be managed (and version-controlled) through Git. And this is made possible by the combination of GitHub Pages and Jekyll.

GitHub Pages, by itself, is pretty simple. It's just a way to host static website files on a GitHub repo. By itself, this would be a pain to setup and manage. And it'd be an even bigger pain to make it look decent.

Enter, Jekyll and Chirpy. Jekyll is a Static Site Generator (SSG), which is essentially able to ingest plain-text like regular old text files and markdown, and output a static website (hence the name). But like all platforms, Jekyll is nothing without some (extensive) configuration. But that's where Chirpy comes in, a Jekyll theme which has done all of the hard work for you. Not only does Chirpy make the resulting website *pretty*, but it makes it *easy* to host on GitHub.

So what's the catch? Weeell, the documentation leaves a bit to be desired for the non-developer. So I've turned my afternoon of frustration into a guide to make this a smoother, less headache-inducing experience for you.

# How to build this site

## Preperations

What you do NOT need:
* A server of any kind
* Business internet which allows you the privilege to pay extra for a static IP
* Comprehensive cybersecurity controls to protect against people hacking your website and getting to your home network

What you do need:
* A GitHub account, and basic familiarity with Git and GitHub
* A custom domain name (optional, but makes it look much nicer)

## Step 1 - Copy the Chirpy Starter Repository

1. Open the Chirpy Starter GitHub repo, <https://github.com/cotes2020/chirpy-starter>
2. Do NOT fork. Click the 'Use this template' button, then 'Create a new repository' to create a brand new repo based on this template
   1. Don't enable 'Include all branches'
   2. The name of your new repository is **very important**. It must be named `<your GitHub username, all lowercase>.github.io` e.g. 'tonysperez.github.io'. This specific name tells GitHub you want this repo to be your GitHub Page repo.
   3. If you have a free GitHub account, you must make your new repo Public (that's right, you can view the [repo](https://github.com/tonysperez/tonysperez.github.io) this site is running out of!). If you have a paid account you should have the ability to make it Private, but that's not something I've tried.
   4. Create Repository!

## Step 2 - Stand up your GitHub Page!

1. Open your brand-new repo
2. Open your repo's Settings
3. In the 'Code and Automation' section, open the 'Pages' tab
4. Under 'Build and Deploy', change 'Source' to `GitHub Actions`
5. Navigate back to your repo's Code, select and edit the file `_config.yml`
6. For now, add your GitHub Page URL to line ~26 (`url: "https://<your GitHub username, all lowercase>.github.io"`).
7. Save and commit this change
8. Navigate to your repo's Actions tab, where you should see two workflows. They will either both be complete (green checks) or still running. They should only take a minute or two to run.
   1. As an aside, it's these Actions which are preforming all the Jekyll magic here. GitHub is just hosting some markdown files and .ymls, but the Actions are what puts all that through Jekyll to get a pretty website out the other side.
9. Check that your GitHub page has been created! Open your GitHub Page link in a web browser, and you should see you have a brand-new (if a bit bare) website!
10. That's if, you now have a free website you don't have to worry about hosting yourself!

## Step 3 - Using Your Custom Domain (optional)

> Proceed with common sense. Messing up your DNS records may cause other services you host on this domain to become inaccessable.
{: .prompt-warning }

### Create the Required DNS Records

1. Log in to your domain registrar, and navigate to the page to manage your domains DNS
2. Decide on what subdomain you want to use to point to your new website. You can use your apex domain (tonystech.com) and www (www.tonystech.com, GitHub will automatically setup both if you choose either), or a different subdomain (blog.tonystech.net, portfolio.tonystech.net, docs.tonystech.net, etc)
   1. If you decide on the domain apex and www
      1. Depending on your registrar, you may or may not be able to create CNAME records at the domain apex.
         1. If you can create a CNAME record at the domain apex, create a CNAME record that points to your GitHub Page URL
         2. If you cannot create a CNAME record at the domain apex, create 4 A and 4 AAA records pointing to GitHub via IP as per [GitHub's documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)
      2. Create a CNAME on www which points to your GitHub Page URL
   2. If you decide on a subdomain, create a CNAME record that points to your GitHub Page URL
3. Wait for your DNS changes to propagate. Some domain registrars will be faster than others (I'm looking at you, GoDaddy and NetworkSolutions) Use the `nslookup` or `dig` utility to query your domain to see if the records have propagated yet.

### Configure your GitHub Page

1. Once the records have propagated, open the Settings on your GitHub Page repo
2. Again in the 'Code and Automation' section, open the 'Pages' tab
3. Near the bottom of that section, there is a section called 'Custom Domain'. Enter the your custom domain into the field, then click 'Save'
4. GitHub will verify that your DNS records are setup correctly
5.  If it just shows 'DNS Check in Progress', it's not actually doing anything. If you wait for a minute and it still only says that, refresh the page
6.  When it's actually checking your DNS, it will open a red box that says something to the effect of 'Checking your DNS'
7. Assuming your DNS is configured correctly, the 'DNS Check in Progress' will be replaced by 'DNS check successful!'. But we're not quite done yet
8. Now, we wait. The HTTPS certificate which GitHub generated for your GitHub Page only includes github.io, it does not include your custom domain. This means that GitHub now has to generate a HTTPS certificate for your custom domain. You don't actually have to do anything here, but this process can take a couple of hours.
9. Once GitHub has generated, issued, and distributed an HTTPS certificate just for your website, the 'Enforce HTTPS' box under the DNS check should become clickable. Click it.
10. Now once again, we wait for GitHub to do it's thing.

### Configure Chirpy

1. Finally, open your repo's Code, and edit `_config.yml` again
2. Update the `url=""` to have your new custom domain URL (https://tonystech.net)
3. Commit this change, and wait a few minutes for the Action to complete and for GitHub to propagate the update
4. Check that you're able to access your GitLab Page using your custom domain. If you can access your Page via your GutHub Page URL but not your custom domain, then you should double-check your DNS records.
5. Done!

## Resources
<https://chirpy.cotes.page/posts/getting-started/>  
<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>  
<https://github.com/cotes2020/jekyll-theme-chirpy>  
<https://github.com/cotes2020/chirpy-starter>  
