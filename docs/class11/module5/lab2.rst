Modify DNS Profile
###############################

Now that we have a Validating Resolver configured with trust anchors, we can enable it by altering our exisiting DNS profile.

Navigate to: **DNS  ››  Delivery : Profiles : DNS**

.. image:: /class2/media/modify_dns_profile_activate_validating_resolver.png

Select the profile "example.com_dns_profile"

Modify the DNS profile to activate the new validating-resolver_cache.

.. image:: /class2/media/modify_dns_profile_validating_resolver.png


.. admonition:: TMSH

   tmsh modify ltm profile dns example.com_dns_profile cache validating-resolver_cache
