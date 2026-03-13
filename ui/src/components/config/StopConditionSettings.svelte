<script>
  import Checkbox from '$lib/components/ui/checkbox.svelte';
  import Input from '$lib/components/ui/input.svelte';
  import Label from '$lib/components/ui/label.svelte';
  import { Percent, Upload, Download, Clock, Users, Pause } from '@lucide/svelte';

  let {
    stopAtRatioEnabled = $bindable(false),
    stopAtRatio = $bindable(2.0),
    randomizeRatio = $bindable(true),
    ratioRangePercent = $bindable(10),
    stopAtUploadedEnabled = $bindable(false),
    stopAtUploadedGB = $bindable(10),
    stopAtDownloadedEnabled = $bindable(false),
    stopAtDownloadedGB = $bindable(10),
    stopAtSeedTimeEnabled = $bindable(false),
    stopAtSeedTimeHours = $bindable(24),
    idleWhenNoLeechers = $bindable(false),
    idleWhenNoSeeders = $bindable(false),
    completionPercent = 100,
    disabled = false,
    onchange,
  } = $props();

  let isLeecherMode = $derived(completionPercent < 100);

  // Derive the effective range based on the target ratio and variance percent
  let ratioRangeInfo = $derived.by(() => {
    if (!stopAtRatioEnabled || !randomizeRatio || !stopAtRatio) return null;
    let base = parseFloat(stopAtRatio);
    let variance = parseFloat(ratioRangePercent);
    if (isNaN(base) || isNaN(variance)) return null;

    let varianceDecimal = variance / 100;
    let min = Math.max(0.01, base * (1 - varianceDecimal)).toFixed(2);
    let max = (base * (1 + varianceDecimal)).toFixed(2);
    return { min, max };
  });
</script>

<div class="bg-muted/50 rounded-lg border border-border overflow-hidden">
  <!-- Ratio -->
  <div
    class="flex items-center gap-3 p-3 border-b border-border {stopAtRatioEnabled
      ? 'bg-primary/5'
      : ''}"
  >
    <Checkbox
      id="stop-ratio"
      checked={stopAtRatioEnabled}
      {disabled}
      onchange={checked => {
        stopAtRatioEnabled = checked;
        onchange?.({ stopAtRatioEnabled: checked });
      }}
    />
    <Percent size={16} class={stopAtRatioEnabled ? 'text-primary' : 'text-muted-foreground'} />
    <Label for="stop-ratio" class="flex-1 cursor-pointer text-sm font-medium">Target Ratio</Label>
    {#if stopAtRatioEnabled}
      <div class="flex items-center gap-1">
        <Input
          type="number"
          bind:value={stopAtRatio}
          {disabled}
          min="0.1"
          max="100"
          step="0.1"
          class="w-20 h-8 text-center font-medium"
          placeholder="2.0"
          oninput={() => onchange?.({ stopAtRatio })}
        />
      </div>
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>

  {#if stopAtRatioEnabled}
    <div class="px-3 pb-3 pt-1 border-b border-border bg-primary/5 pl-[2.25rem]">
      <div class="flex flex-col gap-2 p-2 bg-background/50 rounded-md border border-border/50">
        <div class="flex items-center gap-2">
          <Checkbox
            id="randomize-ratio"
            checked={randomizeRatio}
            {disabled}
            class="h-3.5 w-3.5"
            onchange={checked => {
              randomizeRatio = checked;
              onchange?.({ randomizeRatio });
            }}
          />
          <Label for="randomize-ratio" class="text-xs cursor-pointer text-muted-foreground">
            Randomize target (±{ratioRangePercent}%)
          </Label>
        </div>

        {#if randomizeRatio}
          <div class="flex items-center gap-3 pl-5">
            <input
              type="range"
              bind:value={ratioRangePercent}
              {disabled}
              min="1"
              max="50"
              class="flex-1 h-1.5 bg-muted rounded-lg appearance-none cursor-pointer accent-primary"
              oninput={() => onchange?.({ ratioRangePercent })}
            />
            <Input
              type="number"
              bind:value={ratioRangePercent}
              {disabled}
              min="1"
              max="50"
              class="w-14 h-6 text-xs text-center px-1"
              oninput={() => onchange?.({ ratioRangePercent })}
            />
            <span class="text-xs text-muted-foreground">%</span>
          </div>

          {#if ratioRangeInfo}
            <div class="pl-5 pt-1 text-[10px] text-muted-foreground">
              Instances will stop between <strong>{ratioRangeInfo.min}</strong> and
              <strong>{ratioRangeInfo.max}</strong>
            </div>
          {/if}
        {/if}
      </div>
    </div>
  {/if}

  <!-- Uploaded -->
  <div
    class="flex items-center gap-3 p-3 border-b border-border {stopAtUploadedEnabled
      ? 'bg-primary/5'
      : ''}"
  >
    <Checkbox
      id="stop-uploaded"
      checked={stopAtUploadedEnabled}
      {disabled}
      onchange={checked => {
        stopAtUploadedEnabled = checked;
        onchange?.({ stopAtUploadedEnabled: checked });
      }}
    />
    <Upload
      size={16}
      class={stopAtUploadedEnabled ? 'text-stat-upload' : 'text-muted-foreground'}
    />
    <Label for="stop-uploaded" class="flex-1 cursor-pointer text-sm font-medium">Max Upload</Label>
    {#if stopAtUploadedEnabled}
      <div class="flex items-center gap-1">
        <Input
          type="number"
          bind:value={stopAtUploadedGB}
          {disabled}
          min="0.1"
          step="0.1"
          class="w-20 h-8 text-center font-medium"
          placeholder="10"
          oninput={() => onchange?.({ stopAtUploadedGB })}
        />
        <span class="text-xs text-muted-foreground w-6">GB</span>
      </div>
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>

  <!-- Downloaded -->
  <div
    class="flex items-center gap-3 p-3 border-b border-border {stopAtDownloadedEnabled
      ? 'bg-primary/5'
      : ''}"
  >
    <Checkbox
      id="stop-downloaded"
      checked={stopAtDownloadedEnabled}
      {disabled}
      onchange={checked => {
        stopAtDownloadedEnabled = checked;
        onchange?.({ stopAtDownloadedEnabled: checked });
      }}
    />
    <Download
      size={16}
      class={stopAtDownloadedEnabled ? 'text-stat-download' : 'text-muted-foreground'}
    />
    <Label for="stop-downloaded" class="flex-1 cursor-pointer text-sm font-medium"
      >Max Download</Label
    >
    {#if stopAtDownloadedEnabled}
      <div class="flex items-center gap-1">
        <Input
          type="number"
          bind:value={stopAtDownloadedGB}
          {disabled}
          min="0.1"
          step="0.1"
          class="w-20 h-8 text-center font-medium"
          placeholder="10"
          oninput={() => onchange?.({ stopAtDownloadedGB })}
        />
        <span class="text-xs text-muted-foreground w-6">GB</span>
      </div>
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>

  <!-- Seed Time -->
  <div
    class="flex items-center gap-3 p-3 border-b border-border {stopAtSeedTimeEnabled
      ? 'bg-primary/5'
      : ''}"
  >
    <Checkbox
      id="stop-seedtime"
      checked={stopAtSeedTimeEnabled}
      {disabled}
      onchange={checked => {
        stopAtSeedTimeEnabled = checked;
        onchange?.({ stopAtSeedTimeEnabled: checked });
      }}
    />
    <Clock size={16} class={stopAtSeedTimeEnabled ? 'text-stat-ratio' : 'text-muted-foreground'} />
    <Label for="stop-seedtime" class="flex-1 cursor-pointer text-sm font-medium">Seed Time</Label>
    {#if stopAtSeedTimeEnabled}
      <div class="flex items-center gap-1">
        <Input
          type="number"
          bind:value={stopAtSeedTimeHours}
          {disabled}
          min="0.1"
          step="0.1"
          class="w-20 h-8 text-center font-medium"
          placeholder="24"
          oninput={() => onchange?.({ stopAtSeedTimeHours })}
        />
        <span class="text-xs text-muted-foreground w-6">hrs</span>
      </div>
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>

  <!-- Idle when No Leechers -->
  <div
    class="flex items-center gap-3 p-3 border-b border-border {idleWhenNoLeechers
      ? 'bg-primary/5'
      : ''}"
  >
    <Checkbox
      id="idle-no-leechers"
      checked={idleWhenNoLeechers}
      {disabled}
      onchange={checked => {
        idleWhenNoLeechers = checked;
        onchange?.({ idleWhenNoLeechers: checked });
      }}
    />
    <Pause size={16} class={idleWhenNoLeechers ? 'text-purple-500' : 'text-muted-foreground'} />
    <Label for="idle-no-leechers" class="flex-1 cursor-pointer text-sm font-medium">
      Idle when no leechers
    </Label>
    {#if idleWhenNoLeechers}
      <span class="text-xs text-purple-500 font-medium">0 KB/s</span>
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>

  <!-- Idle when No Seeders -->
  <div class="flex items-center gap-3 p-3 {idleWhenNoSeeders ? 'bg-primary/5' : ''}">
    <Checkbox
      id="idle-no-seeders"
      checked={idleWhenNoSeeders}
      {disabled}
      onchange={checked => {
        idleWhenNoSeeders = checked;
        onchange?.({ idleWhenNoSeeders: checked });
      }}
    />
    <Users size={16} class={idleWhenNoSeeders ? 'text-orange-500' : 'text-muted-foreground'} />
    <Label for="idle-no-seeders" class="flex-1 cursor-pointer text-sm font-medium">
      Idle when no seeders
    </Label>
    {#if idleWhenNoSeeders}
      {#if !isLeecherMode}
        <span class="text-xs text-orange-500 font-medium" title="Only works when completion < 100%"
          >0 KB/s</span
        >
      {:else}
        <span class="text-xs text-orange-500 font-medium">0 KB/s</span>
      {/if}
    {:else}
      <span class="text-xs text-muted-foreground">disabled</span>
    {/if}
  </div>
</div>
