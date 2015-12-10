## How to elegantly maintain a patched `dwm` in Archlinux

### Repository structure

Like many dwm users, I like to apply a couple patches from
dwm.suckless.org/patches to the stock dwm. I also have my own settings and
parameters that I like. To make it easy to keep these patches and settings
up-to-date with the current version or master branch of dwm, each patch has its
own branch. I treat my personal settings as an additional branch. For example,
if the patch branches are based on a current version tagged `6.0`, then the
patchset on top of this will be called `qguv-6.0`:

         ,-- better-borders --,
        /                      \
    6.0 ---- settings ---------- qguv-6.0
        \                      /
         '-- gaps ------------'

Let's say upstream tags a new version `6.1`. To update the patchset to a new
version, start by rebasing each branch to the new tip. We'll be going from this:

          ,--- 6.1
         /
        /  ,-- better-borders --,
       /  /                      \
    6.0 ------ settings ---------- qguv-6.0
        \                        /
         '---- gaps ------------'

to this:

          ,--- qguv-6.0
         /
        /        ,-- better-borders
       /        /
    6.0 --- 6.1 ---- settings
                \
                 '-- gaps

Now make a merge commit merging all the patch branches and tag the result
`qguv-6.1`:

          ,--- qguv-6.0
         /
        /        ,-- better-borders --,
       /        /                      \
    6.0 --- 6.1 ---- settings ---------- qguv-6.1
                \                      /
                 '-- gaps ------------'

#### Example: updating to new tagged version

Here is the above example (updating from 6.0 to 6.1) in long form. Assume the
`patchset` branch is currently pointing to the same commit as the 6.0 tag.

```bash
git fetch --tags upstream

git checkout better-borders
git rebase 6.1
# fix problems etc.

git checkout gaps
git rebase 6.1

git checkout settings
git rebase 6.1

git checkout patchset
git merge --ff-only settings
git merge better-borders
git merge gaps

git tag -m "6.1 with qguv's patchset" qguv-6.1
```

#### Example: updating to new master

Even if upstream hasn't pushed out a new official version, we can still update
to the latest master. The steps are very similar. Again, we assume that
`patchset` is the tip where all the patch branches were last merged.

```bash
git fetch --tags upstream master

git checkout better-borders
git rebase upstream/master
# fix problems etc.

git checkout gaps
git rebase upstream/master

git checkout settings
git rebase upstream/master

# point patchset to the current settings commit
git checkout patchset
git reset --hard settings

# merge and update
git merge better-borders
git merge gaps
```

### Using makepkg

Grab the PKGBUILD.

    git clone aur.archlinux.org/dwm-qguv-git

Make and install.

    makepkg -firs
