# Scaffold gaps (not from either measurement)

No path alias configured. Imports climb three levels:
'../../../design-system'. The agent followed the existing convention, so this
belongs to the initial scaffold.

No React.lazy anywhere, so the agent never reaches for it. Whether dialogs
should be code-split is a project decision that has never been made. Until it
is written down, the agent keeps guessing from what it sees.

The design system README still says --color-*: initial removes the Tailwind
palette and that bg-blue-500 produces no CSS. That reset was removed, so the
sentence is now false. Nothing checks that docs match code.
