<template>
    <component
        :is="`${tag}`"
        :style="stylesComputed">
            <slot></slot>
        </component>
</template>

<script>
export default {
    /**
     * Vue module «StyleBox» that calculate styles for HTML-tag (cannot be 
     * assigned to other module) in diferent resolution. Is needed if you have
     * many elements on page that create no sense to write many classes in CSS.
     * 
     * @property {string} tag           — To set up tag. Have fallback div.
     * @property {object} styles        — Contain default styles. Avoid using inline styles they will override this script creates.
     * @property {array} styleQueries   — Array of objects, which contain media query resolution or root variable, direction and styles.
     */
    name: 'StyleBox',
    props: {
        tag: {
            type: String,
            default: 'div'
        },
        styles: {
            type: Object,
            default() {
                return {};
            }
        },
        styleQueries: Array,
    },
    data() {
        return {
            windowResize: 1
        }
    },
    created() {
        window.addEventListener("resize", this.incrementWindowResize);
    },
    destroyed() {
        window.removeEventListener("resize", this.incrementWindowResize);
    },
    computed: {
        filteredStyleQueries() {
            /**
             * Function that filter styles matched with media queries
             * 
             * @return {array} filteredStyleQueries — Array of user rules. 
             * Styles object included in it.
             */
            return this.styleQueries ? this.styleQueries.filter(query => {
                let querySize = '';
                if(query.size.endsWith('px')) querySize = query.size;
                else {
                    querySize = getComputedStyle(document.documentElement).getPropertyValue(`${query.size}`);
                }
                    
                if(query.direction && query.direction == 'down'){
                    if(window.matchMedia(`(max-width: ${querySize})`).matches){
                        return this.windowResize;
                    }
                } else {
                    if(window.matchMedia(`(min-width:${querySize})`).matches){
                        return this.windowResize;
                    }
                }

                return this.windowResize * 0;
            }) : [];
        },
        stylesFromQueries() {
            /**
             * This functions creates new array from filteredStyleQueries which
             * contains only styles rules.
             * 
             * @return {array} stylesFromQueries — Array of styles.
             */
            let styles = [];
            this.filteredStyleQueries.forEach(query => {
                styles.push(query.styles);
            })
            return styles;
        },
        stylesComputed() {
            /**
             * Creating of collection all styles fits to mediaqueries.
             * 
             * @return {object} stylesComputed — Compunded finished styles.
             */
            return Object.assign({}, this.styles, ...this.stylesFromQueries)
        },
    },
    methods: {
        incrementWindowResize(){
            /**
             * Method that autiincrementing each time user change screen
             * resolution (rotate/debug) for auto recompute style rules.
             */
            this.windowResize += 1;
        }
    }
}
</script>