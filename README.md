This is an isolated example of an issue I'm seeing with how Astro 0.21+ handles CSS scoping when `@keyframe` is used.

## Summary

Using `@keyframes` in a `<style>` block in an astro component breaks selector scoping for all rules after the `@keyframes` instance.

There are two examples in this repo that have the same code.

- One using astro `0.20.x`
- One using astro `0.21.x`
- Each example has its build artifacts committed to git

Having both shows the change between the two and why this seems like a bug.

### Example

Is this example, the `.my-para` declaration will be scoped properly. The `.another-para` will not be scoped.

```
<style>
  .my-para {
    color: blue;
  }

  @keyframes fadeIn {
    to {
      opacity: 100%;
    }
  }

  .another-para {
    color: red;
  }
</style>
```

That input CSS produces the following bundled CSS. **Note**: I've modified formatting here for readability

```
.my-para.astro-FNZOVGOV {color:#00f}
@keyframes fadeIn{to{opacity:100%}}
.another-para{color:red}
```

The expecation is that `.another-para` would be scoped `.another-para.astro-FNZOVGOV`. This was was the behavior in Astro pre 0.21.

## Further testing

From what I've been able to tell, the `@keyframes` rule breaks scoping for all rules that come after it. So, if we modify our above example to put `@keyframes` first, no styles are scoped

```
<style>
  @keyframes fadeIn {
    to {
      opacity: 100%;
    }
  }

  .my-para {
    color: blue;
  }

  .another-para {
    color: red;
  }
</style>
```

produces the following (again, formatting modified here for readability)

```
@keyframes fadeIn{to{opacity:100%}}
.my-para {color:#00f}
.another-para{color:red}
```

## Working around this

This is a bit flimsy, but you can work around this by putting all `@keyframes` rules at the bottom of any `<style>`.

```
<style>
  .my-para {
    color: blue;
  }

  .another-para {
    color: red;
  }

  @keyframes fadeIn {
    to {
      opacity: 100%;
    }
  }
</style>
```

Produces:

```
.my-para.astro-FNZOVGOV {color:#00f}
.another-para.astro-FNZOVGOV {color:red}
@keyframes fadeIn{to{opacity:100%}}
```

## Other @-rules

This seems to be isolated to `@keyframes`. I tested with `@supports` and `@font-face`. Neither of those caused this same, or other, issues.

My hunch is that the extra curly braces needed in `@keyframes` declarations trips things up.
