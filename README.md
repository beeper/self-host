# Self-Host Beeper

![image](https://user-images.githubusercontent.com/1048265/215031408-dc1a7ed0-e119-4d87-a91d-18ceae44f49a.png)


### Beeper is a universal chat app. It’s a single app to chat with friends on 15 different chat networks. Learn more on [beeper.com](http://beeper.com).
 ---

If you're just looking to install Beeper, you can get it [here](http://beeper.com). If you're a developer wanting to contribute or if you want to self-host Beeper on your own infrastructure, read on.

## How Beeper works

---

*Read the full [FAQ](https://beeper.com/faq)*

Beeper is built on top of an open source chat protocol called [Matrix](https://matrix.org). When we started building Beeper, we made a fundamental decision to open source the majority of our backend code and provide you with the option to self-host Beeper. We chose to do this because:
- We want you to be able to inspect the source code that Beeper uses to connect to other chat networks
- We want to contribute to the Matrix ecosystem
- We want to provide great libraries for others to develop open source Matrix bridges


Our infrastructure stack is composed of:

**Beeper Clients**

- Native [iOS](https://www.beeper.com/download) and [Android](https://play.google.com/store/apps/details?id=com.beeper.chat) clients (closed source forks of [Element iOS](https://github.com/vector-im/element-ios) and [Android](https://github.com/vector-im/element-android))
- [Mac OS](https://www.beeper.com/download), [Windows](https://www.beeper.com/download) and [Linux clients](https://www.beeper.com/download) (closed source forks of [Element Web](https://github.com/vector-im/element-web)/[Desktop](https://github.com/vector-im/element-desktop))

**Matrix Homeserver**

- [Synapse](https://github.com/matrix-org/synapse)


*********Open Source, Beeper-maintained Matrix bridges*********
| [mautrix/whatsapp](https://github.com/mautrix/whatsapp) | [mautrix/signal](https://github.com/mautrix/signal) |
| --- | --- |
| [mautrix/telegram](https://github.com/mautrix/telegram) | [mautrix/facebook](https://github.com/mautrix/facebook) |
| [mautrix/iMessage](https://github.com/mautrix/imessage) | [mautrix/twitter](https://github.com/mautrix/twitter) |
| [android-sms](https://gitlab.com/beeper/android-sms) | [mautrix/discord](https://github.com/mautrix/discord) |
| [mautrix/slack](https://github.com/mautrix/slack) | [mautrix/instagram](https://github.com/mautrix/instagram) |
| [hifi/heisenbridge](https://github.com/hifi/heisenbridge) | [mautrix/googlechat](https://github.com/mautrix/googlechat) |
| [beeper/linkedin](https://github.com/beeper/linkedin) | [beeper/groupme](https://github.com/beeper/groupme) |

*********Open source, community maintained Matrix bridges, Beeper sponsored*********

| [fair/kakaotalk](https://src.miscworks.net/fair/matrix-appservice-kakaotalk.git) | [fair/line](https://src.miscworks.net/fair/matrix-puppeteer-line.git) |
| --- | --- |

What is a Matrix bridge? [Learn more](www.beeper.com/new-faqs/what-is-a-bridge)


## Self-Host Install Guide

---

Self hosting Beeper is possible, but not an easy task right now. It requires experience and/or willingness to learn Linux system administration. 

***************************************************************************************************************************************************************************************************************************************Note: only open source Matrix clients like Element or SchildiChat can connect to a self-hosted system at this time. Beeper clients require a Beeper account and use Beeper’s infrastructure.***************************************************************************************************************************************************************************************************************************************

### Instructions (easiest path)

<aside>
💡 We recommend using the https://github.com/spantaleev/matrix-docker-ansible-deploy playbook to deploy Beeper’s Matrix bridges alongside a Matrix homeserver and client

</aside>

1. **Purchase a domain**
    1. We recommend Google Domains
    2. Insert this domain (eg `selfhostbeeper.com`) wherever `<insert_domain>` is shown in these instructions.
2. **Select a hosting provider**
    1. We recommend a 2GB RAM 50GB disk Digital Ocean Droplet
    2. Record the Droplet IP to use in next step
3. **Configure DNS**
    
    
    | Type | Host | Priority | Weight | Port | Target |
    | --- | --- | --- | --- | --- | --- |
    | A | matrix | - | - | - | droplet-ip |
    - Test your domain to make sure it’s pointing at the Droplet IP
        - `nslookup -type=A matrix.<insert_domain>`
4. Install necessary tools on your desktop/laptop 
    
    ************Mac OS************
    `brew install git pwgen`
    ****************Linux****************
    `apt install git pwgen`
    
5. **Download playbook to your desktop/laptop** 
    
    `git clone https://github.com/spantaleev/matrix-docker-ansible-deploy.git && cd matrix-docker-ansible-deploy` 
    
6. Configure the installation
`mkdir inventory/host_vars/matrix.<insert_domain>`
7. `wget https://raw.githubusercontent.com/beeper/self-host/main/vars.yml`
8. `cp vars.yml inventory/host_vars/matrix.<insert_domain>/vars.yml`
9.  open`inventory/host_vars/matrix.<insert_domain>/vars.yml` in a text editor
    1. Insert your `<insert_domain>` in line 12
    2. Enter a real email address in line 27 and 40
    3. Wherever `<create_secretkey>` shows up (eg line 22), switch back to your terminal, run `pwgen -s 64 1` and insert that key into the file
    4. Disable any bridges you do not want (saves RAM!) or enable Telegram by following instructions on Line 52
    5. Save the file.
10. `cp examples/hosts inventory/hosts`
11. Open `inventory/hosts` in a text editor
    1. Replace `<your_domain>` with your domain
    2. Replace `your-server's external IP address>` with Droplet IP. The hosts file shold now look like this:
   ```
   [matrix_servers]
    matrix.YOUR.DOMAIN ansible_host=YOUR_SERVERS_IP ansible_ssh_user=root
   ```
12. Install docker and make sure it is started
    
    Mac OS - Follow instructions [https://docs.docker.com/desktop/install/mac-install](https://docs.docker.com/desktop/install/mac-install) 
    
    Linux - Follow instructions [https://docs.docker.com/desktop/install/linux-install/#generic-installation-steps](https://docs.docker.com/desktop/install/linux-install/#generic-installation-steps)
    
13. Run Ansible in docker from the `/matrix-docker-ansible-deploy` directory. If this doesn't work, check the [playbook instructions](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/ansible.md#using-ansible-via-docker).
    
    ```bash
    docker run -it --rm \
    -w /work \
    -v `pwd`:/work \
    -v $HOME/.ssh/id_rsa:/root/.ssh/id_rsa:ro \
    --entrypoint=/bin/sh \
    docker.io/devture/ansible:2.13.6-r0-1
    ```
    If you use a key type other than RSA, or for any other reason your key file is located elsewhere, you will need to change the above command to point to the correct key.
    
14. Your terminal should now show `/work`, then issue these commands
    1. `git config --global --add safe.directory /work`
    2. `make roles`
    3. `ansible-playbook -i inventory/hosts setup.yml --tags=install-all,ensure-matrix-users-created,start`
    4. Your server is now installed! Check that everything is working `ansible-playbook -i inventory/hosts setup.yml --tags=self-check`
        1. ignore the errors about federation
    - Additional configuration [https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook.md#other-configuration-options](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook.md#other-configuration-options)
    - Note that if you remove components from `vars.yml`, or if we switch some component from being installed by default to not being installed by default anymore, you'd need to run the setup command with `--tags=setup-all` instead of `--tags=install-all`. See [Playbook tags introduction](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/installing.md#playbook-tags-introduction)****
    
    e. Create your user account, change the command to include your preferred username (`<insert_username`) and password (`<your_password>`). 
    
    `ansible-playbook -i inventory/hosts setup.yml --extra-vars='username=<insert_username> password=<your_password> admin=yes' --tags=register-user`
    
15. In a browser, open our recommended Matrix client, [https://app.schildi.chat/#/login](https://app.schildi.chat/#/login) 
    1. Click ‘Edit’ and `https://matrix.<your_domain>` into the homeserver field, eg `https://matrix.beeptest.org` then click Continue
    2. Sign in with your username/password created in step 14e
16. Set up each bridge

    ![CleanShot 2023-01-26 at 23 08 40](https://user-images.githubusercontent.com/1048265/215031550-61f92954-6936-42af-bb4b-a8165e17389e.gif)
    
    1. Click the + icon, then type in `@whatsappbot:<your_domain>` into the search bar and wait for a suggestion to appear, then click on it, then click Done. This will start a chat with the ‘bridge bot’ and provide you with a console to login and setup the bridge
    2. Click the Whatsapp Bridge Bot chat in the left panel, then send the message `help` in the chat to see the list of commands. Each bridge has a slightly different sign in command, but it is usually `login`, `link`, or `login-qr`
    3. Repeat Step 16 for each bridge you would like to configure (eg `@instagrambot:<your_domain>`)
    4. Great! Now all your bridges are set up
17. Set up mobile apps
- Android - [SchildiChat](https://play.google.com/store/apps/details?id=de.spiritcroc.riotx)
- iOS - Recommended apps: [Element](https://apps.apple.com/us/app/element-messenger/id1083446067) or [FluffyChat](https://apps.apple.com/us/app/fluffychat/id1551469600?platform=iphone) 
    
18. Advanced and optional! Enable federation
- Add a root `A record` to your DNS pointing at your Droplet IP
- Modify `vars.yml` to `matrix_nginx_proxy_base_domain_serving_enabled: true`, then re-run the Ansible script
- Confirm it’s working at [https://federationtester.matrix.org/](https://federationtester.matrix.org/)
- How to upgrade your installation later
    - [https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/maintenance-upgrading-services.md#upgrading-the-matrix-services](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/maintenance-upgrading-services.md#upgrading-the-matrix-services)

## Feedback

---

We would love to hear what you think about Beeper and our open source software. We hang out at #beeper:beeper.com on Matrix, or create an issue in this repository. 
