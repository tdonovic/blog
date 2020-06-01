+++
author = "tdonovic"
date = 2020-05-26
title = "secret management, terraform, vault"
series = "secret mangement"
+++

Storing secrets in git is bad. Github even has ([automated secret
scanning)[https://help.github.com/en/github/administering-a-repository/about-secret-scanning]
to try and combat this. The worst bit is having years of security debt,
like secrets in git, and needing to put them somewhere more secure. This
post details some of the trials we have gone through to fix this issue,
as well as our approach and why we think it was a good way to go.

# iac
`infrastructure as code` is the standard way to deploy infrastructure,
since it's repeatable, clean, and can be source controlled. There are a
few tools to do this with, but I would argue the defacto standard for
this is to use Hashicorp's mostly excellent
[terraform](https://terraform.io). Unfortunately, sometimes this means
that people can (inadvertently?) put secrets into the same repo as the
infrastructure. This is a security problem, but it's a big issue for
opening up control of infrastructure to the rest of an org: the thinking
going, we can't give access to our infra repos to devs if there are
secrets in there. If we can let devs into that repo, they can work on
the infrastructure too, meaning we have less to worry about and can
focus on more important things 🤔. Let's do that, that sounds like a
good use of my time.

# removing secrets, aka. down the `vault` rabbit hole
Again, thanks Hashicorp, the defacto standard way to store secrets when
you have any volume of them, and want to be cross platform is to use
[vault](https://www.vaultproject.io/). Even better, `terraform` has a
`vault` provider, so *they just work*<sub>it never does, right?</sub>.

Setting up vault in a secure, HA, good way is a challenge in and of
itself, and a a separate blogpost 😉

Once `vault` is set up, the first thought is, now I need some access
controlled way to auth to it, so that I'm not using the root token in my
`terraform`, which I would argue is worse than leaving the secrets in
the repo to start.

But this is confusing, since everyone I talked to said you need to use
the root token, because `terraform` does unnatural things with it, like
generatign a child token with it. It's scary when you start looking,
start seeing github issues with titles like ["Cannot craete child token
without root
privileges"](https://github.com/terraform-providers/terraform-provider-vault/issues/368)
or ["provider is unable to access data without a non-root
token"](https://github.com/hashicorp/terraform/issues/16457). Time to
get reading and find some solution then.

# a child, non-root, token. for your health
So, a child token. At first inspection, yes, you can absolutely do
this. Just need to proivide the following policy to that entity:
  path "auth/token/create" {
  capabilities = ["create", "update"]
  }
But does that mean that the new, child, token, will be able to do
whatever it wants? No, `vault` has a nice inheritance model, so it won't
give child tokens more permission than their parents.

### side note; why does tf force us to do it this way
  Warning: the tfstate is caching the results, access to vault is only made once.
Terraform caches that token in its statefile. If it uses the actual
token, we have an issue. But with a temp token, we can store it and it's
useless. A nice workaround for a problem that should arguably not be
there.

Nice, but how do we actually give access to this token in the first
place? Great, it's not root, but we still need to store it somewhere

# github auth, sso magic, token helper
To do anything in `vault`, you need to auth. This can be done by `vault
login`, which, irrespective of the auth method, returns a token that is
actually used to do *stuff* in `vault`. 
Great, so what?
Lets just use this token instead of some static token for our
`terraform`. Even better, `vault` stores it in its token helper,
which terraform can also use. Lets go one step futher, and use github as our auth method
for logging in, rather than with some generic iam role. Great, now we
have audit on who got access to `vault` and made a bunch of changes.

# now what
Now lets see if I can get all that working
