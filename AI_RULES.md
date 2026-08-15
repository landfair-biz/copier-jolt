# Application Guidelines

- Build the frontend with **React** and **TypeScript**; keep all application source code in `src/`.
- Use **React Router** for client-side navigation and define routes centrally in `src/App.tsx`.
- Place route-level views in `src/pages/`, with the default page in `src/pages/Index.tsx`.
- Place reusable, app-specific UI in `src/components/`; compose new components into the relevant page so they are visible in the app.
- Use **Tailwind CSS** for all styling, layout, spacing, responsive behavior, and visual states. Avoid separate CSS files unless a third-party integration requires one.
- Prefer existing **shadcn/ui** components for standard interface controls such as buttons, forms, dialogs, menus, cards, tabs, and inputs. Do not edit the installed shadcn component source; wrap or compose it in app components instead.
- Use **Radix UI** primitives only when a needed accessible interaction is not already provided by shadcn/ui.
- Use **lucide-react** for interface icons. Do not add custom icon libraries or inline SVGs when a suitable Lucide icon exists.
- Keep dependencies minimal: use the libraries already installed before adding a package, and add a new dependency only when it is necessary for the requested feature.
