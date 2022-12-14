<!--
Copyright: Ankitects Pty Ltd and contributors
License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<script lang="ts">
    import Col from "../components/Col.svelte";
    import Container from "../components/Container.svelte";
    import Row from "../components/Row.svelte";
    import Spacer from "../components/Spacer.svelte";
    import * as tr from "../lib/ftl";
    import {
        Decks,
        Generic,
        ImportExport,
        importExport,
        Notetypes,
    } from "../lib/proto";
    import DeckSelector from "./DeckSelector.svelte";
    import DelimiterSelector from "./DelimiterSelector.svelte";
    import DupeResolutionSelector from "./DupeResolutionSelector.svelte";
    import FieldMapper from "./FieldMapper.svelte";
    import Header from "./Header.svelte";
    import HtmlSwitch from "./HtmlSwitch.svelte";
    import { getColumnOptions, getCsvMetadata } from "./lib";
    import NotetypeSelector from "./NotetypeSelector.svelte";
    import Preview from "./Preview.svelte";
    import StickyFooter from "./StickyFooter.svelte";
    import Tags from "./Tags.svelte";

    export let path: string;
    export let notetypeNameIds: Notetypes.NotetypeNameId[];
    export let deckNameIds: Decks.DeckNameId[];
    export let dupeResolution: ImportExport.CsvMetadata.DupeResolution;

    export let delimiter: ImportExport.CsvMetadata.Delimiter;
    export let forceDelimiter: boolean;
    export let forceIsHtml: boolean;
    export let isHtml: boolean;
    export let globalTags: string[];
    export let updatedTags: string[];
    export let columnLabels: string[];
    export let tagsColumn: number;
    export let guidColumn: number;
    export let preview: Generic.StringList[];
    // Protobuf oneofs. Exactly one of these pairs is expected to be set.
    export let notetypeColumn: number | null;
    export let globalNotetype: ImportExport.CsvMetadata.MappedNotetype | null;
    export let deckId: number | null;
    export let deckColumn: number | null;

    let lastNotetypeId = globalNotetype?.id;
    let lastDelimeter = delimiter;

    $: columnOptions = getColumnOptions(
        columnLabels,
        preview[0].vals,
        notetypeColumn,
        deckColumn,
        guidColumn,
    );
    $: getCsvMetadata(path, delimiter, undefined, isHtml).then((meta) => {
        columnLabels = meta.columnLabels;
        preview = meta.preview;
    });
    $: if (globalNotetype?.id !== lastNotetypeId || delimiter !== lastDelimeter) {
        lastNotetypeId = globalNotetype?.id;
        lastDelimeter = delimiter;
        getCsvMetadata(path, delimiter, globalNotetype?.id).then((meta) => {
            globalNotetype = meta.globalNotetype ?? null;
            tagsColumn = meta.tagsColumn;
        });
    }

    async function onImport(): Promise<void> {
        await importExport.importCsv(
            ImportExport.ImportCsvRequest.create({
                path,
                metadata: ImportExport.CsvMetadata.create({
                    dupeResolution,
                    delimiter,
                    forceDelimiter,
                    isHtml,
                    forceIsHtml,
                    globalTags,
                    updatedTags,
                    columnLabels,
                    tagsColumn,
                    guidColumn,
                    notetypeColumn,
                    globalNotetype,
                    deckColumn,
                    deckId,
                }),
            }),
        );
    }
</script>

<Container class="csv-page">
    <Row --cols={2}>
        <Col --col-size={1} breakpoint="md">
            <Container>
                <Header heading={tr.importingFile()} />
                <Spacer --height="1.5rem" />
                <DelimiterSelector bind:delimiter disabled={forceDelimiter} />
                <HtmlSwitch bind:isHtml disabled={forceIsHtml} />
                <Preview {columnOptions} {preview} />
            </Container>
        </Col>
        <Col --col-size={1} breakpoint="md">
            <Container>
                <Header heading={tr.importingImportOptions()} />
                <Spacer --height="1.5rem" />
                {#if globalNotetype}
                    <NotetypeSelector
                        {notetypeNameIds}
                        bind:notetypeId={globalNotetype.id}
                    />
                {/if}
                {#if deckId}
                    <DeckSelector {deckNameIds} bind:deckId />
                {/if}
                <DupeResolutionSelector bind:dupeResolution />
                <Tags bind:globalTags bind:updatedTags />
            </Container>
        </Col>
        <Col --col-size={1} breakpoint="md">
            <Container>
                <Header heading={tr.importingFieldMapping()} />
                <Spacer --height="1.5rem" />
                <FieldMapper {columnOptions} bind:globalNotetype bind:tagsColumn />
            </Container>
        </Col>
    </Row>
    <StickyFooter {path} {onImport} />
</Container>

<style lang="scss">
    :global(.csv-page) {
        --gutter-inline: 0.25rem;

        :global(.row) {
            // rows have negative margins by default
            --bs-gutter-x: 0;
            // ensure equal spacing between tall rows like
            // dropdowns, and short rows like checkboxes
            min-height: 3em;
        }
    }
</style>
