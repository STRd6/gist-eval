Gist Eval
=========

Load and evaluate code from a Gist.

    global.PACKAGE = PACKAGE

    Github = require "github"

    api = Github().api

    module.exports = GistEval =
      exec: (id) ->
        api("gists/#{id}")
        .then (data) ->
          files = data.files

          # Eval all .js
          Object.keys(files).forEach (name) ->
            if name.match(/\.js$/)
              Function(files[name].content)()

          # TODO: Build/Eval based on a manifest if present

          # TODO: Sandboxed context where we require a module from a gist

    GistEval.exec(8849828)

Not sure if we need this yet, but we may when we allow full packages to live in
gists.

    packageWrapper = (pkg, code) ->
      """
        ;(function(PACKAGE) {
        var oldRequire = window.Require;
        #{PACKAGE.dependencies.require.distribution.main.content}
        var require = Require.generateFor(PACKAGE);
        window.Require = oldRequire;
        #{code}
        })(#{JSON.stringify(pkg, null, 2)});
      """