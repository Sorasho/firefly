<script lang="typescript">
    import { AccountActionsModal, DashboardPane } from 'shared/components'
    import { getContext } from 'svelte'
    import type { Readable } from 'svelte/store'
    import { AccountActions, AccountBalance, AccountHistory, AccountNavigation, BarChart, LineChart } from '.'
    import { Locale } from 'shared/lib/typings/i18n'
    import { AccountMessage, WalletAccount } from 'shared/lib/typings/wallet'

    export let locale: Locale

    export let onSend = (..._: any[]): void => {}
    export let onInternalTransfer = (..._: any[]): void => {}
    export let onGenerateAddress = (..._: any[]): void => {}

    export let isGeneratingAddress

    const account = getContext<Readable<WalletAccount>>('selectedAccount')
    const accountTransactions = getContext<Readable<AccountMessage[]>>('accountTransactions')

    const viewableAccounts = getContext<Readable<WalletAccount[]>>('viewableAccounts')

    let navAccounts
    $: navAccounts = $account
        ? $viewableAccounts.map(({ id, alias, color }) => ({ id, alias, color, active: $account.id === id }))
        : []

    let showActionsModal = false

    const handleMenuClick = () => {
        showActionsModal = !showActionsModal
    }
</script>

<!-- wait for account to load -->
{#if $viewableAccounts && $account}
    <div class="w-full h-full flex flex-col flex-nowrap p-10 pt-0 relative flex-1 bg-gray-50 dark:bg-gray-900">
        <AccountNavigation {locale} accounts={navAccounts} />
        {#key $account}
            <div class="w-full h-full grid grid-cols-3 gap-x-4 min-h-0">
                <DashboardPane classes=" h-full flex flex-auto flex-col flex-shrink-0">
                    <AccountBalance
                        {locale}
                        color={$account.color}
                        balance={$account.rawIotaBalance}
                        balanceEquiv={$account.balanceEquiv}
                        onMenuClick={handleMenuClick} />
                    <DashboardPane classes="h-full -mt-5 z-0">
                        <AccountActions {isGeneratingAddress} {onSend} {onInternalTransfer} {onGenerateAddress} {locale} />
                    </DashboardPane>
                </DashboardPane>
                <DashboardPane>
                    <AccountHistory {locale} color={$account.color} transactions={$accountTransactions} />
                </DashboardPane>
                <div class=" flex flex-col space-y-4">
                    <DashboardPane classes="w-full h-1/2">
                        <LineChart {locale} />
                    </DashboardPane>
                    <DashboardPane classes="w-full h-1/2">
                        <BarChart {locale} />
                    </DashboardPane>
                </div>
            </div>
        {/key}
        <AccountActionsModal bind:isActive={showActionsModal} {locale} />
    </div>
{/if}
