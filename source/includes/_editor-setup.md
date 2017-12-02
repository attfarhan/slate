
# Editor setup

1. Download and install the editor from [https://about.sourcegraph.com/beta/201708/](/beta/201708).
1. Move Sourcegraph Editor into your Applications folder and open it.
1. On the editor homepage, click `Sign in or sign up to Sourcegraph`. You will be directed to the user settings page on Sourcegraph.com in your web browser. Copy your session ID to your clipboard:

    <img src="/docs/editor/setup/editor_sign_in.gif" alt="Editor Sign-in" />

1. If you are the first Sourcegraph Editor user in your organization, click `Create new organization` in the sidebar. Afterward, invite teammates to your organization by adding their email addresses.

    <img src="/docs/editor/setup/CreateNewOrg.png" alt="Create New Org" />

1. Go back to Sourcegraph Editor and paste in the session ID you just copied.
1. Connect to your code host. By doing this, you are enabling the editor to read all the code you have access to:
    - GitHub.com:
        - Click `Add GitHub Token`. You'll be directed to a page on GitHub.com. Create a GitHub token with repo scope, copy the token to your clipboard, and paste it into your editor.

    <img src="/docs/editor/setup/add_github_token.gif" alt="Add GitHub token" />

    - GitHub Enterprise:
        - use `⇧⌘P + "organization"` and select "Preferences: Open Organization Settings". Set "github.url": `<your GitHub Enterprise domain>`. In a browser, visit `<your GitHub Enterprise domain>`/settings/tokens/new, create a GitHub token with repo scope, copy the token to your clipboard. In your editor, click "Add GitHub Token" and paste the token into your editor.

    - Bitbucket:
        - Click `Set Bitbucket app password`. Under "Access Management", select "App passwords". Create a new app password with full repositories scope (read, write, admin, delete). Copy the token to your clipboard and paste it into your editor after entering your Bitbucket user name.


1. If you're a VS Code user: use `⇧⌘P + "migrate"` and select `Migrate Extensions and User Settings from Visual Studio Code` to migrate your extensions and settings from VS Code to Sourcegraph Editor
1. Configure Slack to notify you each time a comment is made in your organization.
    - Visit `<your-workspace-url>`.slack.com/apps/new/A0F7XDUAZ-incoming-webhooks
    - Create a new channel called Sourcegraph > refresh the page > choose the Sourcegraph channel > Add Incoming Webhooks integration
    - Copy the Slack webhook URL and paste it into the Slack webhook URL input box on https://sourcegraph.com/settings/ `<your-Sourcegraph-org>`
1. See [https://about.sourcegraph.com/products/editor/](/products/editor) for usage docs

## Enterprise users

Enterprise users should configure their editor to connect to their private Sourcegraph Server instance:

1. Open your editor user settings (⇧⌘P + "user settings").
1. Add `"remote.endpoint": "http://localhost:3080"` to the settings JSON file and save.
1. Sign your editor in to your Sourcegraph Server instance (⇧⌘P + "sign in sourcegraph", copy the session token from the page opened in your web browser).

If you have any questions, email [support+editor@sourcegraph.com](mailto:support+editor@sourcegraph.com).
