<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Cause } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import type { Report } from '#lib/kb/causes';

  // A merged ticket split into one ticket per cause, counted the desk's way (causes.ts as exported).
  type Split = { url: string; outcome: string };
  let { ticket, splits = {} }: { ticket: number; splits?: Partial<Record<Cause | 'unplaced', Split>> } = $props();

  const all = reports as Report[];
  const merged = $derived(uniqueById(all).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const issueNo = (url: string) => `#${url.split('/').pop()}`;
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> {groups.length === 1 ? 'problem' : 'problems'}
  <span class="muted">· the export holds {all.length} rows; {all.length - merged.length} repeat an id already counted</span>
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>outcome</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const s = splits[g.cause]}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if s}<a href={s.url}>{issueNo(s.url)}</a>{:else}<span class="warn">none yet</span>{/if}</td>
        <td>{s?.outcome ?? ''}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="muted">{moved} of the <code>{g.cause}</code> reports read as a one-day move; counted under <code>{g.cause}</code> by tenant.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85rem; }
  .warn { color: var(--warn); font-weight: 600; }
</style>
