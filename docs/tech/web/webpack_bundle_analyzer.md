# webpack-bundle-analyzer

- [https://github.com/webpack-contrib/webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)

1.

```
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';

...
new BundleAnalyzerPlugin({
  analyzerMode: "static",
  openAnalyzer: false,
  reportFilename: `../report-${target}.html`
})
```

2.

```
yarn webpack --json > stats.json
yarn webpack-bundle-analyzer stats.json
```
