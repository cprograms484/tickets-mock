<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem the reports show. The desk's one-day moves are their own problem,
  // not the cause their tenant would give them (desk-notes.json).
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const problemOf = (r: Report) => (notes.oneDayMove.includes(r.id) ? 'move' : (causeOf(r, tenantOf(r)) ?? 'unplaced'));

  const all = $derived(reports as Report[]);
  const merged = $derived(uniqueById(all).filter((r) => r.mergedInto === ticket));
  const rows = $derived(
    [...Map.groupBy(merged, problemOf).entries()]
      .map(([problem, rs]) => {
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        return {
          problem,
          reports: rs.length,
          tenants: new Set(rs.map((r) => r.tenant)).size,
          firstSeen: days[0],
          lastSeen: days[days.length - 1],
        };
      })
      .sort((a, b) => b.reports - a.reports),
  );
</script>

<p class="sum">
  <strong>{all.length}</strong> rows in the export · <strong>{merged.length}</strong> reports once each ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code></td>
        <td>{r.reports}</td>
        <td>{r.tenants}</td>
        <td>{r.firstSeen} → {r.lastSeen}</td>
        <td>{#if issues[r.problem]}<a href={issues[r.problem]}>#{issues[r.problem].split('/').pop()}</a>{:else}none{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
