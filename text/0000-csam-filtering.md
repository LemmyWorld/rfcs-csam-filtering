---
Feature Name: csam_filtering
Start Date: 2023-09-12
RFC PR: (coming soon)[LemmyNet/rfcs#0000](https://github.com/LemmyNet/rfcs/pull/0000)
---

# csam_filtering

## Summary

This feature wants to implement a way to combat CSAM / unwanted images.

## Motivation

This feature wants to reduce the liability risk of instance operators and fight against CSAM in its many kinds and forms. The mere possibility of running into CSAM on the Fediverser creates great mental stress for everyone involved. Even brief exposure to this content has caused both admins and moderators to immediately quit, for both legal and ethical reasons.
More about it here <https://github.com/iftas-org/resources/blob/main/CSAM-CSE/README.md>

## Guide-level explanation

As an admin, you can add your credentials to different CSAM filtering Services ( For Example [Microsoft's PhotoDNA](https://www.microsoft.com/en-us/photodna)) in the Admin Settings and/or activate an offline CSAM scanner ( For example [AI Horde csam scanner](https://github.com/Haidra-Org/horde-safety/blob/main/horde_safety/csam_checker.py), [Thorn Safer [paid]](https://get.safer.io/csam-detection-tool-for-child-safety), [Meta PQD](https://github.com/facebook/ThreatExchange/tree/main/pdq)).

In theory, only one CSAM filtering Service or offline CSAM scanner is required.

- Ideal GUI actions that the moderator should be able to take upon detecting CSAM:

  - [ ] Remove the Image from the storage
  - [ ] Don't federate Posts/Comments/DMs before the CSAM check is done
  - [ ] Ban the inflicting user.
  - [ ] Remove Data from the inflicting User
  - [ ] Purge the inflicting user.
  - [ ] Notify Admins over dms [ Local and if possible non-local admins ]
  - [ ] Pass on Error ( If the checks somehow errored out after a few retries, should it still get through? )
  - [ ] Lazy Checking ( This will activate lazy checking the image => Instead of blocking the upload, it "allows" the image through but it will purge the comment/post/dm and purge the image from storage afterward if it hits a check )

- You can activate each options then with a checkbox and it will scan all <u>**new**</u> images and log its actions ( Thumbnails, Embeds, Images in Posts, Images in Comments, Images in private messages). The scanning of existing images should be a separate piece of systems automation.

## Reference-level explanation

- The picture upload process has to change if this feature is active:

  - For Every upload or image creation ( from local or federated ) it has to go through activated checks before it finally lands in the storage.

  - The checks have to be done before showing that post/comment/dm.

  - If the checks failed because it returned positive for CSAM, the user and his post/comment/dm will then have the options enabled in the admin panel.

  - If the checks failed because of an error it should either let it through or block it, depending on a toggle setting (ideally in the admin GUI).

  - The checks should be scalable so that it doesn't create lag on a large instance. Best case, it should cache hashes of already known bad images of previous checks. Some of the services have "Bulk" image uploads, those should be used if there is such an option for checking uploads in sets.

- Database Changes:

  - Create a table "CSAM-detection-status" with the columns

    | Field      | Type | Requied | Notes                                                                             |
    | ---------- | ---- | ------- | --------------------------------------------------------------------------------- |
    | Id         | UUID | true    | Primary Key                                                                       |
    | UserID     | str  | true    | reference to the users' table                                                     |
    | Status     | bool | false   | only when the status is "done"                                                    |
    | CSAM       | bool | false   | only when the status is "done"                                                    |
    | entityId   | int  | false   | referencing to posts/comments/dms - only used when lazy loading is active         |
    | entityType | str  | false   | either one of "post" \| "comment" \| "dm" - only used when lazy loading is active |

- API Changes:

  - New API for the settings
  - New API for the logs for the admins ( the logs can be deferred from the "CSAM-detection-status" table )

- Federation Changes:

  - This feature would be ONLY local. ( It will still check federated posts )

## Drawbacks

- False Positives (mostly only on Local Filtering as some of them are AI / ML-based, but there is the possibility of Hash Collision on the external Services with Image Hash detection )
- For external Services, there is a reliance on external closed source filtering.
- Adding more complexity on Image Uploads ( and with it a delay )

## Rationale and alternatives

- Why is this design the best in the space of possible designs?
  - Due to there being no other official implementations
- What other designs have been considered and what is the rationale for not choosing them?
  - There are no other integrated designs. Alternative solutions require tying into Pictrs, rather than processing in the Lemmy backend itself as a separate worker process.
- What is the impact of not doing this?
  - Legal liability of admins ( and in some way the developers of Lemmy )
  - Trauma of the users, mods, and admins.
- Could this change be implemented in an external project instead? How much complexity does it add?
  - No, as this should be a core feature of the backend.

## Prior art

- Does this feature exist in other social media platforms and what experience has their community had?
  - Every other social media platform has this feature ( and even made his own [see Meta PQD) because it reduces the stress and liability Risk of Admins.
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.
  - <https://github.com/iftas-org/resources/blob/main/CSAM-CSE/README.md#detection>
  - <https://decoded.legal/blog/2022/11/notes-on-operating-fediverse-services-mastodon-pleroma-etc-from-an-english-law-point-of-view>
  - <https://www.eff.org/deeplinks/2022/12/user-generated-content-and-fediverse-legal-primer>
  - <https://www.europarl.europa.eu/RegData/etudes/BRIE/2022/738224/EPRS_BRI(2022)738224_EN.pdf>
  - <https://denise.dreamwidth.org/91757.html>
  - <https://en.wikipedia.org/wiki/Legality_of_child_pornography>

## Unresolved questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
  - To work out any technical or UI design details
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
  - To review different external scanning tooling that could be injected into the upload pipeline.
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?
  - To look into additional ML related (or non-hash) detection methods

## Future possibilities

- Another good extension of this would be sort of a simple Image Hash filtering. Where admins can add images (hashes) of unwanted "Porn" images and it gets filtered out of any posts/comments/dms, with it repeated assaults can be reduced.
- The most problematic point is that even after purging a post/comment/dm, the image(s) stay in storage and because a ticking liability time bomb. It would be better to never save the image in the first place.
