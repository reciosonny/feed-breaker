<script lang="ts">
    import dayjs from 'dayjs';
    import toast, { Toaster } from 'svelte-french-toast';
    import Switch from "$lib/components/Switch/Switch.svelte";
    import RadioOptions, { type OptionSelection } from "$lib/components/RadioOptions/RadioOptions.svelte";
    import { Tabs, TabsList, Tab, TabsPanel } from "$lib/components/Tabs";
    import Button from "$lib/components/Button/Button.svelte";
    import TextInput from "$lib/components/TextInput/TextInput.svelte";
    import RichTextInput from "$lib/components/RichTextInput/RichTextInput.svelte";
    import QuoteCard from "$lib/components/QuoteCard/QuoteCard.svelte";
    import Pagination from "$lib/components/Pagination/Pagination.svelte";

    import "./options.scss";
    import { getChromeStorage, removeChromeStorage, setChromeStorage } from '$lib/chromeStorage';
    import type { DisableReelOptions, Quote } from '$lib/types';

    let showQuotes = $state(false);
    let showDisableFor = $state(false);
    let disableReelOptions: boolean = $state(false);
    let disableReelOptionsDuration: DisableReelOptions = $state('10 mins.');

    let enableQuotes: boolean = $state(false);
    let quotes: Quote[] = $state([]);
    let showAddForm: boolean = $state(false);
    let editingQuoteId: string | null = $state(null);
    let formQuote: string = $state('');
    let formAuthor: string = $state('');
    let currentPage: number = $state(1);    
    const QUOTES_PER_PAGE = 5;

    const totalPages = $derived(Math.max(1, Math.ceil(quotes.length / QUOTES_PER_PAGE)));
    const paginatedQuotes = $derived(
        quotes.slice((currentPage - 1) * QUOTES_PER_PAGE, currentPage * QUOTES_PER_PAGE)
    );

    // @ts-ignore
    window.dayjs = dayjs;

    const computeDisableDuration = (val: DisableReelOptions) => {
        let disableDuration: number = 0;
        switch (val) {
            case '10 mins.':
                disableDuration = dayjs().add(10, 'm').valueOf();
                break;
            case '30 mins.':
                disableDuration = dayjs().add(30, 'm').valueOf();                
                break;
            case '1 hour':
                disableDuration = dayjs().add(1, 'h').valueOf();
                break;
            case '2 hours':
                disableDuration = dayjs().add(2, 'h').valueOf();
                break;
            case '4 hours':
                disableDuration = dayjs().add(4, 'h').valueOf();
                break;
            case '8 hours':
                disableDuration = dayjs().add(8, 'h').valueOf();
                break;
            default:
                break;
        }

        return disableDuration;
    }

    const setDisableDuration = async (val: DisableReelOptions) => {
        
        if (val === 'forever') {
            await setChromeStorage('DISABLE_DURATION', 'forever');
        } else {
            const disableDuration = computeDisableDuration(val);
            await setChromeStorage('DISABLE_DURATION', disableDuration);
        }

        await setChromeStorage('DISABLE_DURATION_SELECTION', val);
    }

    const onOptionChanged = async (val: DisableReelOptions) => {
        await setChromeStorage('DISABLE_DURATION', val);
        await setDisableDuration(val);
        
        toast.success("Option changed!");
    };

    const reelOptionSelections: OptionSelection<DisableReelOptions>[] = [
        { value: "10 mins.", label: "10 minutes" },
        { value: "30 mins.", label: "30 minutes" },
        { value: "1 hour", label: "1 hour" },
        { value: "2 hours", label: "2 hours" },
        { value: "4 hours", label: "4 hours" },
        { value: "8 hours", label: "8 hours" },
        { value: "forever", label: "forever" },
    ];

    // TODO: Maybe we can mark this as deprecated later
    const initializeLocalStoreValues = async () => { //TODO: How about we move the initialization to a global store instead? What if?
        const storeReelOptions = await getChromeStorage('DISABLE_REEL_OPTIONS');
        disableReelOptions = storeReelOptions;
        showDisableFor = disableReelOptions;

        const storeOptionsDuration = await getChromeStorage('DISABLE_DURATION_SELECTION');
        if (storeOptionsDuration) {
            disableReelOptionsDuration = storeOptionsDuration;
        }
    }

    const initializeQuotesValues = async () => {
        const storedEnableQuotes = await getChromeStorage('ENABLE_QUOTES');
        if (storedEnableQuotes !== null && storedEnableQuotes !== undefined) {
            enableQuotes = storedEnableQuotes;
        }

        const storedQuotes = await getChromeStorage('CUSTOM_QUOTES');
        if (storedQuotes) {
            quotes = storedQuotes;
        }
    }

    const persistQuotes = async (updatedQuotes: Quote[]) => {
        quotes = updatedQuotes;
        await setChromeStorage('CUSTOM_QUOTES', updatedQuotes);
    }

    const onEnableQuotesChange = async (val: boolean) => {
        enableQuotes = val;
        await setChromeStorage('ENABLE_QUOTES', val);
        toast.success('Option changed!');
    }

    const resetForm = () => {
        formQuote = '';
        formAuthor = '';
        showAddForm = false;
        editingQuoteId = null;
    }

    const onSaveQuote = async () => {
        if (!formQuote.trim() || !formAuthor.trim()) {
            toast.error('Please fill in both fields.');
            return;
        }

        if (editingQuoteId) {
            const updatedQuotes = quotes.map(q =>
                q.id === editingQuoteId
                    ? { ...q, quote: formQuote.trim(), author: formAuthor.trim() }
                    : q
            );
            await persistQuotes(updatedQuotes);
            toast.success('Quote updated!');
        } else {
            const newQuote: Quote = {
                id: crypto.randomUUID(),
                quote: formQuote.trim(),
                author: formAuthor.trim(),
            };
            await persistQuotes([...quotes, newQuote]);
            toast.success('Quote added!');
        }

        resetForm();
    }

    const onEditQuote = (quote: Quote) => {
        editingQuoteId = quote.id;
        formQuote = quote.quote;
        formAuthor = quote.author;
        showAddForm = true;
    }

    const onDeleteQuote = async (id: string) => {
        const updatedQuotes = quotes.filter(q => q.id !== id);
        await persistQuotes(updatedQuotes);

        const newTotalPages = Math.max(1, Math.ceil(updatedQuotes.length / QUOTES_PER_PAGE));
        if (currentPage > newTotalPages) {
            currentPage = newTotalPages;
        }

        toast.success('Quote deleted!');
    }

    $effect(() => {
        initializeLocalStoreValues();
        initializeQuotesValues();
    });
    
    const onDisableFor = async (val: boolean) => {
        showDisableFor = val;
        if (!val) {
            await removeChromeStorage('DISABLE_REEL_OPTIONS');
            await removeChromeStorage('DISABLE_DURATION');
            await removeChromeStorage('DISABLE_DURATION_SELECTION');
        } else {
            const disableDuration = computeDisableDuration('10 mins.'); //default value when enabling
            await setChromeStorage('DISABLE_DURATION', disableDuration);
            await setChromeStorage('DISABLE_REEL_OPTIONS', val);
        }
        toast.success('Option changed!');
    }

    const txtFooter = `Made with <3 by Sonny`;
</script>

<main class="px-[250px] pt-12">
    <section class="flex items-center justify-center">
        <h1 class="option-header text-center heading-lg px-[30px] py-[15px] bg-black text-white inline relative">Feed breaker</h1>
    </section>
    <section class="mt-20 box-border py-[30px] px-10 bg-primary-100">
        <h2 class="heading-md text-white">Settings</h2>
        <div>
            <span>lorem ipsum dolor amet - I like this keyboard than ever before</span>
        </div>
        <Tabs defaultValue="settings" class="pt-4">
            <TabsList>
                <Tab value="settings" class="text-neutral-50">Basic</Tab>
                <Tab value="quotes" class="text-neutral-50">Quotes</Tab>
                <Tab value="pomodoro" class="text-neutral-50">Pomodoro</Tab>
            </TabsList>
            <TabsPanel value="settings">
                <section>
                    <Switch
                        id="thisIsId"
                        className="mt-4"
                        onCheckedChange={onDisableFor}
                        checked={showDisableFor}
                    >
                        <span class="text-body-lg font-body font-normal text-white">Disable "For You" blocking</span>
                    </Switch>

                    {#if showDisableFor}
                        <section class="pt-4">
                            <h3 class="text-body-lg font-body text-white">Disable for</h3>
                            <RadioOptions
                                class="pl-3.5 pt-2"
                                value={disableReelOptionsDuration}
                                selections={reelOptionSelections}
                                onValueChanged={onOptionChanged}
                            />
                        </section>
                    {/if}
                </section>
                <span class="text-body-lg font-body pt-[240px] block text-center text-white">{txtFooter}</span>
            </TabsPanel>
            <TabsPanel value="quotes">
                <section>
                    <Switch
                        id="enableQuotes"
                        className="mt-4"
                        onCheckedChange={onEnableQuotesChange}
                        checked={enableQuotes}
                    >
                        <span class="text-body-lg font-body font-normal text-white">Enable quotes</span>
                    </Switch>

                    {#if enableQuotes}
                        <section class="mt-6 w-[592px]">
                            <h3 class="text-body-xl text-white">Your custom quotes</h3>

                            {#if !showAddForm}
                                <Button variant="default" class="mt-4" onclick={() => (showAddForm = true)}>
                                    + Add quote
                                </Button>
                            {:else}
                                <div class="mt-4 w-96">
                                    <label for="authorQuote" class="text-body-lg font-body font-normal text-white block mb-2">Author quote</label>
                                    <RichTextInput id="authorQuote" bind:value={formQuote} placeholder="Type your quote..." rows={4} />

                                    <label for="authorName" class="text-body-lg font-body font-normal text-white block mb-2 mt-4">Author name</label>
                                    <TextInput id="authorName" bind:value={formAuthor} placeholder="Type something..." />

                                    <div class="flex gap-3 mt-4 justify-end">
                                        <Button variant="edit" onclick={onSaveQuote}>
                                            Save changes
                                        </Button>
                                        <Button variant="delete" onclick={resetForm}>
                                            Cancel
                                        </Button>
                                    </div>
                                </div>
                            {/if}

                            {#if quotes.length > 0}
                                <div class="mt-6 flex flex-col gap-2">
                                    {#each paginatedQuotes as q (q.id)}
                                        <QuoteCard
                                            quote={q.quote}
                                            author={q.author}
                                            onEdit={() => onEditQuote(q)}
                                            onDelete={() => onDeleteQuote(q.id)}
                                        />
                                    {/each}
                                </div>

                                <Pagination
                                    {currentPage}
                                    {totalPages}
                                    onPageChange={(page) => (currentPage = page)}
                                />
                            {/if}
                        </section>
                    {/if}
                </section>
            </TabsPanel>
            <TabsPanel value="pomodoro">
                <!-- TODO: Add updates -->
            </TabsPanel>
        </Tabs>
    </section>
</main>

<Toaster containerClassName="font-body font-medium" />