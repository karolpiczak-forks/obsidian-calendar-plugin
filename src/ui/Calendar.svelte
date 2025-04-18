<svelte:options immutable />

<script lang="ts">
  import type { Moment } from "moment";
  import {
    Calendar as CalendarBase,
    configureGlobalMomentLocale,
  } from "obsidian-calendar-ui";
  import type { ICalendarSource } from "obsidian-calendar-ui";
  import { onDestroy } from "svelte";

  import type { ISettings } from "src/settings";
  import { activeFile, dailyNotes, settings, weeklyNotes } from "./stores";

  let today: Moment;

  $: today = getToday($settings);

  export let displayedMonth: Moment = today;
  export let sources: ICalendarSource[];
  export let onHoverDay: (date: Moment, targetEl: EventTarget, isMetaPressed: boolean) => void;
  export let onHoverWeek: (date: Moment, targetEl: EventTarget, isMetaPressed: boolean) => void;
  export let onClickDay: (date: Moment, inNewSplit: boolean) => Promise<void>;
  export let onClickWeek: (date: Moment, inNewSplit: boolean) => Promise<void>;
  export let onContextMenuDay: (date: Moment, event: MouseEvent) => void;
  export let onContextMenuWeek: (date: Moment, event: MouseEvent) => void;

  export function tick() {
    today = window.moment();
  }

  function getToday(settings: ISettings) {
    configureGlobalMomentLocale(settings.localeOverride, settings.weekStart);
    dailyNotes.reindex();
    weeklyNotes.reindex();
    return window.moment();
  }

  // 1 minute heartbeat to keep `today` reflecting the current day
  let heartbeat = setInterval(() => {
    tick();

    const isViewingCurrentMonth = displayedMonth.isSame(today, "day");
    if (isViewingCurrentMonth) {
      // if it's midnight on the last day of the month, this will
      // update the display to show the new month.
      displayedMonth = today;
    }
  }, 1000 * 60);

  onDestroy(() => {
    clearInterval(heartbeat);
  });
</script>

<CalendarBase
  {sources}
  {today}
  onHoverDay={(date, targetEl) => onHoverDay(date, targetEl, false)}
  onHoverWeek={(date, targetEl) => onHoverWeek(date, targetEl, false)}
  onContextMenuDay={(date, event) => { onContextMenuDay(date, event); return true; }}
  onContextMenuWeek={(date, event) => { onContextMenuWeek(date, event); return true; }}
  onClickDay={(date, isMetaPressed) => { onClickDay(date, isMetaPressed); return true; }}
  onClickWeek={(date, isMetaPressed) => { onClickWeek(date, isMetaPressed); return true; }}
  bind:displayedMonth
  localeData={today.localeData()}
  selectedId={$activeFile}
  showWeekNums={$settings.showWeeklyNote}
/>
