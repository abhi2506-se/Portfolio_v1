# Live Chat v44: link previews, message order, video sending, admin light theme, What's new

## Link previews (live chat)
- A link in a message is clickable and opens in a new tab. The text stays selectable,
  so it can be copied.
- When a message with a link is sent, the server fetches the title, description, site
  name, and image once, and stores them with the message. Every reader sees the same
  card. Fetches are SSRF-safe (http/https only, private and loopback addresses blocked,
  up to 3 redirects, 4 second timeout, 512 KB read limit).
- The link preview adds up to 4 seconds to sending a link message, because the send
  waits for the preview. A preview that can't be fetched just isn't shown.

## Message order
- Media (photo, video, voice) keeps the time it was picked, not the time it finished
  uploading. A video that takes a minute to compress no longer lands after the texts
  sent during that minute.

## Video sending
- Vercel limits request bodies to about 4.5 MB. Videos over 3.5 MB are compressed in
  the browser to fit under 4 MB, with 480p for clips over 2 minutes.
- A video that's still over 4 MB after compression is not sent. The panel shows
  "This video is too long to send, even after compression…".
- Upload and compression errors now show under the composer for 6 seconds. They used
  to fail silently.

## Admin chat light/dark toggle
- A ☀/🌙 button sits in the corner of the admin chat. The choice is saved in the
  browser under `admin-chat-theme`, which is separate from the visitor side.
- The theme is scoped to `.admin-chat-light`, so it changes only the admin chat. The
  visitor's chat stays dark, and the visitor's theme never reads the admin's.

## What's new (live)
- Public: a "What's new" button sits bottom-left on the site (hidden on /admin and
  /portal). Each time it's opened, it fetches the latest entries from the database
  with no cache, so new entries show up without a deploy.
- Admin: add and delete entries at **/admin/whats-new** (admin login required).
- Table `site_updates` is created automatically on first use.

## Files
- New: `lib/link-preview.ts`, `lib/site-updates-db.ts`, `app/api/whats-new/route.ts`,
  `app/admin/whats-new/page.tsx`, `components/whats-new-button.tsx`,
  `components/live-chat/link-preview.tsx`, this file.
- Changed: `lib/live-chat-db.ts` (`createdAt` option, `link_preview` column,
  `setMessageLinkPreview`), `app/api/live-chat/route.ts` (`sentAt`, link previews),
  `lib/video-compress.ts` (4 MB target), `components/live-chat/message-bubble.tsx`
  (clickable links, preview card, scoped class for own bubbles),
  `components/live-chat/user-chat-panel.tsx`, `admin-chat-panel.tsx` (send time,
  errors, admin theme), `app/layout.tsx` (mounts the What's new button).

## Known limits
- Videos longer than about 3–4 minutes can still come out of compression too blurry
  or too large. Sending bigger files would need direct-to-storage upload, which isn't
  in this version.
- A text message that fails to send is still shown as failed, with no retry button
  yet.
- The admin theme remaps the panel's colors through CSS. A new color class added
  later may need its own mapping.

## Blocked chat continues after unblock (v45)
- A block never closes the chat. The gate remembers the blocked chat's id in
  localStorage (`__lc_blocked_chat`), so the chat comes back even if the visitor
  returns a day later after an unblock, or after a reload.
- A visitor who closed the blocked chat with the password doesn't get it back on unblock.

## What's new admin page is protected (v46)
- `/admin/whats-new` now redirects to the admin login when there's no valid admin session.
  The POST and DELETE API routes were already admin-only.

## Admin chat light theme fixed (v47)
- The admin chat's light mode now flips the whole panel with a CSS filter (`invert` +
  `hue-rotate`), so text and backgrounds always contrast. Images and videos are flipped back.
- Overlays in the admin panel (confirm dialogs, wallpaper picker, extra security,
  auto-delete, archive, banner) render through `Portal` into `document.body`. Filters
  would otherwise break their fixed positioning, and they keep their own dark look.
- The admin choice is saved only under `admin-chat-theme`. The visitor's chat never reads it.

## Blocked chat: password hides the notice, it doesn't end the chat (v48)
- Only the visitor or the admin ends a chat. Entering the chat password on the block
  screen only hides that notice on this device. The chat stays open.
- The blocked notice shows no message history or previews.
- Unblock reopens the chat unless the visitor or the admin actually ended it.
- Chats that were already closed by an older version: an unblock reopens them, since
  they weren't ended by a user or the admin.

## Visitor chat: options menu and light/dark theme (v49)
- The header keeps the avatar, the name (in full width), the presence line, and a ✕.
  Everything else is in the ⋮ menu: voice and video calls, chat media, wallpaper, chat
  history, auto-delete, light/dark mode, expand/shrink (desktop), hide chat, and end chat.
- Light/dark mode is set by scoped CSS variables on the chat's root element. The site's
  own theme is never changed. The choice is stored under `visitor-chat-theme`, separate
  from the admin's `admin-chat-theme`. The chat is dark by default.

## Avatar, unread badges, screen-aware notifications (v50)
- Visitor chat avatar: the real admin shows the Journey main profile photo. A co-admin
  shows their first letter. While Extra Security hides the admin, a bot icon shows.
- Admin list badges: a visitor's unread messages show a green count. A new request
  (pending chat) shows a red "New request" tag.
- Push notifications for visitor messages are sent only when the admin doesn't have that
  chat open and visible (presence `in_chat` within the last 20 seconds). New request
  notifications are unchanged.
- Shared image/video click: not reproduced from code. The server logic and viewer look
  correct. The exact error text from the browser console is needed to pin it down.

## Admin chat layout fix (v51)
- On mobile, the admin chat panel is fixed to the full screen. A `relative` class was added
  to the panel root for the theme button, and Tailwind applies `relative` after `fixed`. That
  stopped the panel from being fixed, so the chat box grew with its messages instead of
  scrolling. `relative` now applies only to the desktop layout.
