# MULTI Research Group Website - Core Design

## Overview

The MULTI (Machine learning, Uncertainty visualization, Large-scale visualization, Tensor visualization, and Inverse problem) research group website is a single-page site showcasing the team and their publications. The site is part of the SCI Institute at the University of Utah.

## Design Philosophy

### Data-Interface Separation

The fundamental principle is **complete separation between content data and rendering logic**. This enables:

- Non-technical maintainers to update content by editing JSON files
- No risk of breaking the site when adding/removing people or publications
- Clear ownership: data files for content, code files for behavior

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        index.html                           │
│  (Structure only - no hardcoded content except boilerplate) │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         /css/                               │
│  variables.css  - Design tokens (colors, fonts, spacing)    │
│  main.css       - All styles (layout, components, pages)    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                          /js/                               │
│  main.js        - Data loading, rendering, interactions     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        /data/                               │
│  group.json        - Group name, description, intro text    │
│  team.json         - Array of team members                  │
│  publications.bib  - Publications in BibTeX format          │
└─────────────────────────────────────────────────────────────┘
```

## Page Sections

### 1. Header Banner
- University of Utah logo (links to utah.edu)
- SCI Institute logo (links to sci.utah.edu)
- Group name: "MULTI Research Group"
- Fixed position, always visible

### 2. Group Introduction
- Brief paragraph explaining MULTI's research focus
- Content loaded from `data/group.json`

### 3. Team Carousel
- Horizontal scrollable/sliding cards
- Each card shows: photo, name, position, affiliation
- Clicking a card:
  - Highlights the selected member
  - Shows member detail card
  - Scrolls to member detail section
  - Filters publications by that author
- Clicking selected card clears filter

### 4. Member Detail Card
- Research banner image (visualization highlight)
- Photo, name, full titles list
- Contact links (email, personal website)
- Background paragraph (supports HTML links)
- Research interests as tags
- Hidden by default, shown when team card clicked

### 5. Publications List
- Filterable by author (via team card selection)
- Searchable by title/keyword/year
- Sorted by year (newest first)
- Each entry shows: title, authors, venue, year
- Links: PDF, Copy BibTeX, DOI

## Data Flow

```
Page Load
    │
    ├─► Load group.json ──► Render intro section
    │
    ├─► Load team.json ──► Render team cards
    │
    └─► Load publications.bib ──► Parse BibTeX ──► Render publication list
                                        │
User Clicks Team Card                   │
    │                                   │
    └─► Filter publications by author ◄─┘
            │
            └─► Re-render publication list (filtered)
```

## File Structure

```
/multi-website/
├── index.html              # Development shell (loads content.html)
├── content.html            # Embeddable content for WordPress
├── /css/
│   ├── variables.css       # CSS custom properties
│   └── main.css            # All styles
├── /js/
│   └── main.js             # All JavaScript
├── /data/
│   ├── group.json          # Group description
│   ├── team.json           # Team members
│   └── publications.bib    # Publications in BibTeX format
├── /images/
│   ├── /team/              # Team member headshots
│   ├── /research/          # Research visualization banners
│   └── /logos/             # University and SCI logos
└── /docs/                  # This documentation
```

## Data Schemas

### group.json

```json
{
  "name": "MULTI Research Group",
  "fullName": "Machine learning, Uncertainty visualization, Large-scale visualization, Tensor visualization, and Inverse problem",
  "institution": "Scientific Computing and Imaging (SCI) Institute",
  "university": "University of Utah",
  "introduction": "Paragraph describing the group's research focus..."
}
```

### team.json

```json
{
  "team": [
    {
      "id": "chris-johnson",
      "name": "Chris Johnson",
      "position": "Distinguished Professor, Group Lead",
      "affiliation": "Scientific Computing and Imaging Institute, University of Utah",
      "titles": [
        "Distinguished Professor of Computer Science",
        "Founding Director, SCI Institute"
      ],
      "email": "crj@sci.utah.edu",
      "website": "https://users.cs.utah.edu/~crj/",
      "photo": "images/team/chris-johnson.jpg",
      "researchBanner": "images/research/chris-johnson-banner.jpg",
      "background": "Biography paragraph with optional <a href=\"...\">HTML links</a>.",
      "researchInterests": ["Uncertainty Visualization", "Scientific Computing"],
      "authorVariants": ["C. R. Johnson", "Chris R. Johnson"]
    }
  ]
}
```

The `authorVariants` field is critical - it lists all name variations used in publications for accurate filtering.

### publications.bib

Publications are stored in standard BibTeX format. The JavaScript parses this file client-side.

```bibtex
@Article{SCI:Har2026a,
  author =    "J. Hart and C. R. Johnson",
  title =     "Publication Title",
  journal =   "Journal Name",
  year =      "2026",
  url =       "https://...",
  doi =       "10.xxxx/xxxxx",
}
```

Supported entry types: `@Article`, `@InProceedings`, `@Book`, `@InCollection`, `@Misc`

## Interaction Design

### Team Card Selection
1. User clicks a team member card
2. Card receives "selected" visual state
3. Page smoothly scrolls to publications section
4. Publications filter to show only those with matching author
5. Filter indicator shows "Showing publications by [Name]"
6. "Clear filter" button appears

### Publication Filtering
- Filter matches against `authorVariants` array from team.json
- Case-insensitive matching
- Multiple selection not supported (one author at a time)

### Search
- Real-time filtering as user types
- Searches: title, authors, venue, year
- Debounced (300ms delay) to prevent excessive re-renders

## Responsive Behavior

| Breakpoint | Team Cards | Publications |
|------------|------------|--------------|
| Mobile (<576px) | Vertical stack, swipeable | Single column |
| Tablet (576-992px) | Horizontal scroll | Single column |
| Desktop (>992px) | Horizontal scroll, larger cards | Single column |

## Accessibility

- Skip link to main content
- Keyboard navigation for team cards (arrow keys)
- ARIA labels on interactive elements
- Focus indicators on all clickable items
- Screen reader announcements for filter changes
