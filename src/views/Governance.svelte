<script>
  import { onMount } from 'svelte';
  import { _ } from 'svelte-i18n';
  import ViewContainer from '../components/elements/ViewContainer.svelte';
  import PageHeader from '../components/elements/PageHeader.svelte';
  import ContainerWithHeader from '../components/elements/ContainerWithHeader.svelte';
  import { getOpenProposals, getVotesForAddress } from '../middleware/snapshot';
  import governance from '../stores/governance';
  import settings from '../stores/settings';
  import Button from '../components/elements/Button.svelte';
  import DetailView from '../components/composed/Table/governance/DetailView.svelte';
  import { BarLoader } from 'svelte-loading-spinners';
  import Table from '../components/composed/Table/Table.svelte';
  import HeaderCell from '../components/composed/Table/HeaderCell.svelte';
  import ExpandRowCell from '../components/composed/Table/ExpandRowCell.svelte';
  import IpfsCell from '../components/composed/Table/governance/IpfsCell.svelte';
  import SnapshotCell from '../components/composed/Table/governance/SnapshotCell.svelte';
  import GenericCell from '../components/composed/Table/governance/GenericCell.svelte';
  import StatusCell from '../components/composed/Table/governance/StatusCell.svelte';

  const openAllOnSnapshot = () => {
    window.open('https://snapshot.org/#/alchemixstakers.eth', '_blank');
  };
  const openDiscussions = () => {
    window.open('https://forum.alchemix.fi/public/t/aip', '_blank');
  };

  /*
   * @dev transforms snapshot's block notation to human readable date
   * @param snapshotBlock the block number
   * */
  const snapshotToDate = (snapshotBlock) => {
    return new Date(snapshotBlock * 1e3).toLocaleDateString($settings.userLanguage.locale);
  };

  // set up columns for table
  const columns = [
    {
      columnId: 'col0',
      value: '',
      colSize: 1,
    },
    {
      columnId: 'col1',
      CellComponent: HeaderCell,
      value: $_('table.title'),
      colSize: 8,
    },
    {
      columnId: 'col2',
      CellComponent: HeaderCell,
      value: $_('table.status'),
      colSize: 2,
    },
    {
      columnId: 'col3',
      CellComponent: HeaderCell,
      value: $_('table.start'),
      colSize: 2,
    },
    {
      columnId: 'col4',
      CellComponent: HeaderCell,
      value: $_('table.end'),
      colSize: 2,
    },
    {
      columnId: 'col5',
      CellComponent: HeaderCell,
      value: $_('table.snapshot'),
      colSize: 3,
    },
    {
      columnId: 'col6',
      CellComponent: HeaderCell,
      value: $_('table.ipfs'),
      colSize: 3,
    },
  ];

  // initialize rows, fill with data later
  let rows = [];

  $: if ($governance.proposals.length > 0) {
    $governance.proposals.forEach((proposal) => {
      const payload = {
        col0: {
          CellComponent: ExpandRowCell,
          expandedRow: {
            ExpandedRowComponent: DetailView,
            proposalEntry: proposal,
          },
          colSize: 1,
        },
        col1: {
          value: proposal.title,
          colSize: 8,
          alignment: 'justify-self-start',
          CellComponent: GenericCell,
          state: proposal.state,
        },
        col2: {
          value: proposal.state,
          colSize: 2,
          CellComponent: StatusCell,
          state: proposal.state,
        },
        col3: {
          value: snapshotToDate(proposal.start),
          colSize: 2,
          CellComponent: GenericCell,
          state: proposal.state,
        },
        col4: {
          value: snapshotToDate(proposal.end),
          colSize: 2,
          CellComponent: GenericCell,
          state: proposal.state,
        },
        col5: {
          snapshot: proposal.snapshot,
          colSize: 3,
          CellComponent: SnapshotCell,
          state: proposal.state,
        },
        col6: {
          ipfsShort: proposal.ipfs.slice(0, 8),
          ipfsId: proposal.ipfs,
          colSize: 3,
          CellComponent: IpfsCell,
          state: proposal.state,
        },
      };
      rows.push(payload);
    });
  }

  onMount(() => {
    if ($governance.proposals.length === 0) {
      getOpenProposals();
      getVotesForAddress();
    }
  });
</script>

<ViewContainer>
  <div class="flex justify-between" slot="head">
    <PageHeader
      pageIcon="alcx_thin.svg"
      pageTitle="{$_('governance_page.title')}"
      pageSubtitle="{$_('governance_page.subtitle')}"
    />
  </div>
  <div class="w-full">
    <p class="text-center text-xs mb-6 opacity-50">
      {$_('governance_page.noTranslation')}
    </p>
    <ContainerWithHeader>
      <div slot="header" class="py-4 px-6 text-sm flex justify-between">
        <!--        <div class="flex space-x-4">-->
        <!--          <Button-->
        <!--            label="{$_('governance_page.all')}"-->
        <!--            width="w-max"-->
        <!--            canToggle="{true}"-->
        <!--            selected="{true}"-->
        <!--            solid="{false}"-->
        <!--            borderSize="0"-->
        <!--          />-->

        <!--          <Button-->
        <!--            label="{$_('governance_page.active')}"-->
        <!--            width="w-max"-->
        <!--            canToggle="{true}"-->
        <!--            selected="{false}"-->
        <!--            solid="{false}"-->
        <!--            borderSize="0"-->
        <!--          />-->

        <!--          <Button-->
        <!--            label="{$_('governance_page.closed')}"-->
        <!--            width="w-max"-->
        <!--            canToggle="{true}"-->
        <!--            selected="{false}"-->
        <!--            solid="{false}"-->
        <!--            borderSize="0"-->
        <!--          />-->
        <!--        </div>-->
        <p>{$_('governance_page.proposals')}</p>

        <div>
          <Button
            label="{$_('governance_page.openOnForum')}"
            borderSize="1"
            height="h-8"
            width="w-max"
            fontSize="text-md"
            on:clicked="{() => openDiscussions()}"
          >
            <svg
              slot="leftSlot"
              class="w-5 h-5"
              fill="currentColor"
              viewBox="0 0 48 48"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M20 32C28.8366 32 36 26.6274 36 20C36 13.3726 28.8366 8 20 8C11.1634 8 4 13.3726 4 20C4 22.6842 5.17509 25.1626 7.16049 27.1616C6.35561 29.4537 5.31284 31.1723 4.6499 32.1319C4.4071 32.4834 4.65714 32.9802 5.08289 32.9453C6.78453 32.8058 10.1224 32.3105 12.3741 30.5519C14.6411 31.4754 17.2389 32 20 32Z"
              ></path>
              <path
                d="M22.7843 33.8337C31.4033 32.7928 38 26.9957 38 20.0002C38 19.4632 37.9611 18.9333 37.8855 18.4121C41.5534 20.1003 44 23.136 44 26.6002C44 28.7476 43.0599 30.7303 41.4716 32.3295C42.068 34.0278 42.8276 35.3325 43.3579 36.1259C43.5953 36.481 43.3423 36.9779 42.917 36.9372C41.5041 36.8021 39.0109 36.3773 37.3007 35.0418C35.4872 35.7806 33.4089 36.2002 31.2 36.2002C27.9781 36.2002 25.0343 35.3074 22.7843 33.8337Z"
              ></path>
            </svg>
          </Button>
          <Button
            label="{$_('governance_page.openAllOnSnapshot')}"
            borderSize="1"
            height="h-8"
            width="w-max"
            fontSize="text-md"
            on:clicked="{() => openAllOnSnapshot()}"
          >
            <svg
              class="w-5 h-5"
              fill="currentColor"
              role="img"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
              slot="leftSlot"
            >
              <title>AMP</title>
              <path
                d="M12 0c6.628 0 12 5.373 12 12s-5.372 12-12 12C5.373 24 0 18.627 0 12S5.373 0 12 0zm-.92 19.278l5.034-8.377a.444.444 0 00.097-.268.455.455 0 00-.455-.455l-2.851.004.924-5.468-.927-.003-5.018 8.367s-.1.183-.1.291c0 .251.204.455.455.455l2.831-.004-.901 5.458z"
              ></path>
            </svg>
          </Button>
        </div>
      </div>

      <div slot="body">
        {#if $governance.fetching}
          <div class="flex justify-center my-4">
            <BarLoader color="{$settings.invertColors ? '#6C93C7' : '#F5C59F'}" />
          </div>
        {:else if $governance.proposals.length > 0}
          <Table rows="{rows}" columns="{columns}" />
        {:else}
          <p class="text-center opacity-50">
            {$_('governance_page.noOpenVotes')}
          </p>
        {/if}
      </div>
    </ContainerWithHeader>
  </div>
</ViewContainer>
