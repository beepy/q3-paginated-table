# Q3 Paginated Table Exercise

Live version at: https://main--cool-eclair-be6f40.netlify.app

Requires Node 16 to build.

Vue front end using (somewhat unnecessarily) Nuxt. I used [Vuetify](https://vuetifyjs.com), which I hadn't actually used before, because it had select and pagination controls which looked close to the original design.

## Notes:

- I changed the format of the returned value from the supplied `getRecommendation` function to include the total number of records, since the api would be able to efficiently supply this number and the example was "fetching" all records anyway. In a real server, that information might instead be included in a response header to keep the result data "pure."
- The sample video seemed to show that the procedure columns also changed between clinics (the first had no SCS column) but there was no data for that in `getNameMappings()` so I elected to just show the SCS column regardless.
- For the purposes of this exercise, I just included some scss to make the Vuetify components look more like the design. There appear to be more proper was to customize it (like setting scss variables) but I did not look deeply into it.

Below is the default Nuxt README:

# Nuxt 3 Minimal Starter

Look at the [Nuxt 3 documentation](https://nuxt.com/docs/getting-started/introduction) to learn more.

## Setup

Make sure to install the dependencies:

```bash
# npm
npm install

# pnpm
pnpm install

# yarn
yarn install
```

## Development Server

Start the development server on `http://localhost:3000`:

```bash
# npm
npm run dev

# pnpm
pnpm run dev

# yarn
yarn dev
```

## Production

Build the application for production:

```bash
# npm
npm run build

# pnpm
pnpm run build

# yarn
yarn build
```

Locally preview production build:

```bash
# npm
npm run preview

# pnpm
pnpm run preview

# yarn
yarn preview
```

Check out the [deployment documentation](https://nuxt.com/docs/getting-started/deployment) for more information.
