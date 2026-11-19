# MMMovies — App ya Android ya kuuza na kutazama movies

Tawi la **MMEmpire / MMMedia**. App imejengwa kwa **Flutter**.

## Kilichomo
- Login / kufungua akaunti / kuingia kama mgeni
- **Nyumbani:** movies maalum (slider) + mistari ya kila kundi (Bongo Movies, Action, Comedy, Tamthilia, Mapenzi, Historia, Watoto, Documentary)
- **Tafuta:** kwa jina, mtayarishaji, au kundi
- **Ukurasa wa movie:** maelezo, bei, umri, trailer, movies zinazofanana
- **Malipo kwa simu:** M-Pesa, Mixx by Yas (Tigo Pesa), Airtel Money, HaloPesa (mteja anaweka PIN kwenye simu)
- **Kukodisha:** ukilipia unatazama kwa saa 48
- **Player:** HLS (.m3u8) na MP4, skrini nzima (landscape)
- **Movies Zangu:** ulizolipia + muda uliobaki
- **Ulinzi:** link ya video inatolewa na server kwa aliyelipia tu

```
mmmovies/
├── lib/
│   ├── main.dart              ← mwanzo wa app
│   ├── config.dart            ← WEKA FUNGUO ZA SUPABASE HAPA
│   ├── theme.dart             ← rangi (nyekundu/nyeusi/nyeupe)
│   ├── models.dart
│   ├── data/                  ← demo + supabase + store
│   ├── screens/               ← skrini zote
│   └── widgets/               ← kadi, poster, dirisha la malipo
├── android/app/src/main/AndroidManifest.xml
└── supabase/
    ├── schema.sql             ← database
    └── functions/             ← malipo (create-payment, payment-webhook)
```

---

## HATUA A — Iendeshe sasa hivi (Demo mode, bila server)

1. Sakinisha **Flutter**: https://docs.flutter.dev/get-started/install/windows/mobile
   (sakinisha pia **Android Studio** + Android SDK). Kisha endesha:
   ```
   flutter doctor
   ```
   Hakikisha "Android toolchain" ina ✓.

2. Fungua folder la `mmmovies` kwenye terminal, kisha tengeneza mafaili ya Android
   yanayokosekana (hayatafuta msimbo wako):
   ```
   flutter create --org com.mmempire --project-name mmmovies --platforms android .
   flutter pub get
   ```

3. Unganisha simu yako (washa **Developer options → USB debugging**) au fungua emulator, kisha:
   ```
   flutter run
   ```

4. **Tengeneza APK** ya kuweka kwenye simu yoyote:
   ```
   flutter build apk --release
   ```
   APK itakuwa: `build/app/outputs/flutter-apk/app-release.apk`

Kwenye Demo mode: email yoyote + password ya herufi 6 inaingia. Malipo ni ya kuigiza
(yanakamilika baada ya sekunde ~4). Video ni za majaribio.

---

## HATUA B — Iunganishe na server halisi (Supabase)

1. Fungua akaunti https://supabase.com → **New project**.
2. **SQL Editor** → bandika `supabase/schema.sql` yote → **Run**.
3. **Project Settings → API**: nakili **Project URL** na **anon public key**,
   kisha uziweke kwenye `lib/config.dart`.
4. **Authentication → Providers**: washa *Email*. Kama unataka "Endelea kama mgeni",
   washa pia *Anonymous sign-ins*.
5. Ongeza movies kupitia **Table Editor**:
   - `movies`: jina, maelezo, bei (`price_tzs`, 0 = bure), poster_url...
   - `movie_sources`: link ya video ya movie hiyo (SIRI)
   - `movie_categories`: unganisha movie na makundi yake
   (Kuna mfano wa SQL mwishoni mwa `schema.sql`.)

## HATUA C — Video (Bunny Stream inapendekezwa)
1. Fungua akaunti https://bunny.net → **Stream** → tengeneza *Video Library*.
2. Pakia movie. Bunny inaitengeneza HLS (360p/720p/1080p) yenyewe.
3. Nakili link ya **HLS playlist** (`.../playlist.m3u8`) → weka kwenye `movie_sources.video_url`.
4. Kwenye Security ya library: washa **Allowed domains / token authentication** ili link isiibiwe.
   (Hatua ya juu zaidi: tengeneza signed URL ndani ya Edge Function, si link ya kudumu.)

## HATUA D — Malipo (AzamPay mfano)
1. Jisajili kama developer https://developers.azampay.co.tz → pata *App Name, Client ID,
   Client Secret, API Key* za **sandbox**.
2. Sakinisha Supabase CLI: https://supabase.com/docs/guides/cli, kisha:
   ```
   supabase login
   supabase link --project-ref <project-ref-yako>
   supabase secrets set AZAMPAY_APP_NAME=... AZAMPAY_CLIENT_ID=... AZAMPAY_CLIENT_SECRET=... AZAMPAY_API_KEY=... AZAMPAY_ENV=sandbox WEBHOOK_SECRET=<neno-refu-la-siri>
   supabase functions deploy create-payment
   supabase functions deploy payment-webhook --no-verify-jwt
   ```
3. Kwenye dashboard ya AzamPay weka **Callback URL**:
   `https://<project-ref>.supabase.co/functions/v1/payment-webhook?secret=<WEBHOOK_SECRET>`
4. Jaribu kwa sandbox. Ukiwa tayari (baada ya usajili wa biashara - BRELA/TIN),
   badilisha `AZAMPAY_ENV=production` na funguo za production.

> Majina ya fields za AzamPay kwenye `functions/` yahakikishwe na nyaraka zao rasmi.
> Ukitumia ClickPesa/Selcom badala yake, badilisha function `requestMobilePush` na
> jinsi webhook inavyosoma callback.

## Jinsi malipo yanavyofanya kazi
```
App → create-payment (Edge Function) → AzamPay → simu ya mteja (PIN)
                                                 ↓
App inaangalia status kila sekunde 3  ←  payment-webhook inaandika "paid" + expires_at
                                                 ↓
          movie_sources inafunguka (RLS) → Player inacheza
```
Bei inachukuliwa kutoka database na server, si kutoka kwenye app, ili mtu asiweze kudanganya bei.

## Kubadilisha muonekano
- Rangi: `lib/theme.dart`
- Jina na muda wa kukodisha: `lib/config.dart`
- Icon ya app: tumia package `flutter_launcher_icons` na logo ya MMEmpire.

## Kabla ya kuweka Play Store
- Tengeneza *upload keystore* na usaini app: https://docs.flutter.dev/deployment/android
- `flutter build appbundle --release` → pakia `.aab` kwenye Google Play Console
- Andaa Privacy Policy (inahitajika na Google Play)

## Logo na icon ya app
Logo iko `assets/logo.png`. Kuifanya pia kuwa icon ya app kwenye simu:
1. Ongeza kwenye `pubspec.yaml` (chini ya dev_dependencies): `flutter_launcher_icons: ^0.14.1`
2. Ongeza mwishoni mwa `pubspec.yaml`:
   ```
   flutter_launcher_icons:
     android: true
     image_path: "assets/logo.png"
     adaptive_icon_background: "#0B0B0B"
     adaptive_icon_foreground: "assets/logo.png"
   ```
3. Endesha: `flutter pub get` kisha `dart run flutter_launcher_icons`
