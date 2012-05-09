# Maxmind country lookup

## Existing solutions

There are a few but they use the city/block tables and also seem to be mysql
specific. As I don't want to try converting a bunch of Polygon code to postgres
format I thought I'd just convert the fairly simple django geoip-redirect code.

## Database setup

Grab this file: <http://geolite.maxmind.com/download/geoip/database/GeoIPCountryCSV.zip>.

SQL for tables:

    CREATE TABLE geoip_country (
        id SERIAL PRIMARY KEY,
        start_ip char(15) NOT NULL,
        end_ip char(15) NOT NULL,
        start_decimal bigint NOT NULL,
        end_decimal bigint NOT NULL,
        country_code varchar(2) NOT NULL,
        country_name varchar(120) NOT NULL
    );

    COPY geoip_country
        (start_ip, end_ip, start_decimal, end_decimal, country_code, country_name)
        FROM '/tmp/GeoIPCountryWhois.csv' DELIMITERS ',' CSV;

I should probably *rake* task all that!

## Usage

In your application controller you can set it to fire as a filter using:

    before_filter :geoip

And then adding to your controller:

    def geoip
        # Lookup the current request IP and assign to session vars, if we've looked
        # it up before then skip further requests and return the session version
        if session[:country_code].nil?
          @country_code = Geoip::Country.find_by_ip(request.remote_ip)
          session[:country_code] = @country_code
        else
          @country_code = session[:country_code]
        end
        return @country_code
    end

This will cache the first lookup to the users session; which is probably how
you'll want to use it for most cases. To use elsewhere expose the method as a
helper in your controller:

    helper_method :geoip
