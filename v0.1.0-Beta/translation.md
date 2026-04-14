# Translation

fxPanel supports 30+ languages for the in-game interface and chat messages.

---

## Supported Languages

Translations cover the in-game menu, warning/ban messages, chat messages, and Discord notification text. The language is set globally in fxPanel Settings.

Arabic, Bulgarian, Bosnian, Czech, Danish, German, Greek, English, Spanish, Estonian, Persian, Finnish, French, Croatian, Hungarian, Indonesian, Italian, Japanese, Lithuanian, Latvian, Mongolian, Nepali, Dutch, Norwegian, Polish, Portuguese, Romanian, Russian, Slovenian, Swedish, Thai, Turkish, Ukrainian, Vietnamese, Chinese

---

## Custom Locales

If your language is not available, or you want to customize messages, you can create a custom locale file.

1. Create a `locale.json` file inside the `txData` folder, based on any existing language file from the repository.
2. Go to fxPanel Settings and select the **"Custom"** language option.
3. Set the `$meta.humanizer_language` key to a language code compatible with the `humanize-duration` library.

> **Quick Testing**
>
> Edit the `locale.json` file and then click "Save Global Settings" in the settings page. Changes take effect immediately without restarting fxPanel or the server.

> **Performance Note**
>
> Custom locales on big servers may have reduced performance due to the way dynamic content is synced to clients. It is strongly encouraged that you contribute translations via GitHub so they get packed with fxPanel.

> **Validation**
>
> To verify your locale file has all required keys, clone the fxPanel repository, run `npm i`, move your `locale.json` into the `locale/` folder, and run `npm run locale:check`.

---

## Contributing

We rely on the community to keep translations updated and high-quality. To contribute a new translation or update an existing one:

1. Create a custom locale file following the instructions above.
2. Name the file using the [ISO 639-1 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) (e.g., `es.json` for Spanish).
3. The `$meta.label` must be the language name in English (e.g., "Spanish" not "Español").
4. Add the language to `shared/localeMap.ts` in alphabetical order.
5. Test your changes in-game and take screenshots as evidence.
6. Submit a Pull Request with the screenshots. An automatic check will run — make sure to read its output in case of any errors.
